# Knowledge Base and Associated Resources

This document describes the **Knowledge Base (KB)** feature in ACE and all **AWS and system resources** associated with it: provisioning (S3, IAM, Bedrock KB and Data Source, optional S3 Vectors), the DocsSync SQS queue, the docs-sync worker, and the sync flow. It does not contain sensitive data (no account IDs, no tokens, no bucket names that reveal client/project identity beyond naming patterns).

---

## What the Knowledge Base is

The Knowledge Base is an **AWS Bedrock**–based feature that stores project documentation (e.g. markdown from a GitHub repo) and makes it queryable for RAG (retrieval-augmented generation) and LLM flows. Each project can have one KB. Resources are **provisioned on demand** by the backend when a project is configured with a documentation repository (`docsRepo`, optional `docsRepoBranch`). Sync (clone repo → upload to S3 → Bedrock ingestion) runs asynchronously via a dedicated SQS queue and worker so that it does not block the API or send output to Slack.

---

## High-level flow

1. **Configuration**: User sets `docsRepo` (and optionally `docsRepoBranch`) for a project in the dashboard. Backend may trigger **ensure KB + enqueue sync** (with debounce).
2. **Provisioning** (ace-stack-backend): If the project has no KB yet, the backend creates (or reuses) in AWS: S3 docs bucket, IAM role for Bedrock, optional S3 Vector bucket and index, Bedrock Knowledge Base, Bedrock Data Source linked to the S3 docs bucket. It then persists `knowledgeBaseId` and `dataSourceId` in the DB via **ace-db-gateway** (`PUT /api/projects/:id/knowledge-base`).
3. **Enqueue sync**: Backend builds a script (clone repo, `aws s3 sync` of `*.md` from **`$WORKDIR/src/docs`** in the cloned repository to the bucket, start Bedrock ingestion job, poll until COMPLETE/FAILED or timeout). The sync path is **hardcoded** in **ace-stack-backend** (`KnowledgeBaseSyncService`). It sends a message to the **DocsSync SQS FIFO queue** with the script and `project_id`. Backend also ensures a **GitHub webhook** on the docs repo for push events → `POST /api/webhooks/docs/:projectId`.
4. **Worker** (ace-commands-api): The **docs-sync worker** (separate process) consumes the DocsSync queue, runs the script (no Slack output), and exits. The script uses GitHub token, repo, branch, bucket name, KB ID, Data Source ID, and AWS region (injected by backend).
5. **Manual sync**: Admin can call **POST /api/admin/knowledge-base/sync** with `{ projectId }` to enqueue a sync without changing config.

**ace-manual layout**: The ACE system documentation repository **ace-manual** stores markdown under **`src/docs/features/`**, not under `src/docs/`. The docs-sync script above uses **`src/docs`** until the backend supports a configurable path; if ace-manual is the configured docs repo for a Knowledge Base, align repository layout with that script (or change the backend) so sync finds the `.md` files.

---

## AWS and system resources

### SQS – DocsSync queue

- **Purpose**: FIFO queue for docs-sync jobs. Decouples the backend from the long-running sync (clone, S3 upload, Bedrock ingestion).
- **Producer**: ace-stack-backend (sends JSON: `commands` array of script lines, `project_id`). Uses `MessageGroupId: docs-sync` and a deduplication ID.
- **Consumer**: ace-commands-api **docs-sync worker** (separate process; not the main HTTP server). Worker needs `QUEUE_DOCS_SYNC_URL` and AWS credentials (S3 write, Bedrock StartIngestionJob).
- **Creation**: Via script **ace-infra/scripts/create-docs-sync-queue.sh** or, when enabled, Terraform module **terraform-library/docs-sync-queue**. Queue name pattern: `<environment>-ace-DocsSync.fifo` (must end with `.fifo`).
- **Env var**: `QUEUE_DOCS_SYNC_URL` in backend and in the docs-sync worker. See [../environments/env-vars-and-secrets.md](../environments/env-vars-and-secrets.md).

### S3 – Documents bucket

- **Purpose**: Stores the markdown (and other) files that are synced from the project’s docs repo. Bedrock Data Source reads from this bucket for ingestion.
- **Creation**: By **ace-stack-backend** (KnowledgeBaseProvisioningService), not by Terraform. Idempotent: if bucket exists, it is reused.
- **Naming pattern**: `<prefix>-kb`. The prefix is derived from client name and project name (sanitized, lowercased, hyphenated); if none, `ace-kb`. Example pattern: `clientname-projectname-kb`.
- **Settings**: Versioning enabled; public access blocked (BlockPublicAcls, IgnorePublicAcls, BlockPublicPolicy, RestrictPublicBuckets). Region: same as backend (`AWS_REGION`, default us-east-1).
- **Sync**: Worker runs `aws s3 sync` from **`$WORKDIR/src/docs`** in the clone (same hardcoded path as in `KnowledgeBaseSyncService`) with `--include "*.md"` (and optionally other patterns) into this bucket.

### AWS Bedrock – Knowledge Base and Data Source

- **Knowledge Base**: Vector KB. Created by backend with:
  - Embedding model: **amazon.titan-embed-text-v2:0** (dimensions 1024, FLOAT32).
  - Storage: **S3 Vectors** (vector bucket and index; see below).
  - IAM role: Bedrock service role created by backend (Bedrock can assume it for reading S3 and writing vectors).
- **Data Source**: S3 type, linked to the **docs bucket** (`<prefix>-kb`). Chunking: **hierarchical** (e.g. 1500 / 300 max tokens, 60 overlap). Created by backend; idempotent by name.
- **Ingestion**: Worker starts an ingestion job via `aws bedrock-agent start-ingestion-job` and polls until COMPLETE/FAILED (max wait e.g. 10 minutes). Bedrock reads from the S3 docs bucket and writes embeddings to the vector store.
- **Region**: Same as backend (us-east-1 by default).

### S3 Vectors (optional)

- **Purpose**: Stores vector embeddings for the Knowledge Base. Used when the KB storage type is S3_VECTORS.
- **Resources**:  
  - **Vector bucket**: Naming pattern `<prefix>-vector-bucket`. Created by backend (S3VectorsClient).  
  - **Index**: Name `default-index` (float32, dimension 1024, euclidean distance, metadata keys for Bedrock text/metadata).
- **Creation**: By backend during provisioning; idempotent.

### IAM

- **Role**: Backend creates an IAM role for Bedrock (trust policy: `bedrock.amazonaws.com`, source account and knowledge-base ARN condition). Naming pattern: `BedrockKBRole_<suffix>` (suffix derived from prefix hash).
- **Policies** (created and attached by backend):  
  - S3 full access (for Bedrock to read docs bucket).  
  - Bedrock full access (for KB operations).  
  - S3 Vectors full access (for vector store).  
- **Path**: `/service-role/`. No account IDs or ARNs are documented here; they are derived at runtime.

### DB Gateway – Persistence

- **API**: **ace-db-gateway** exposes:
  - `PUT /api/projects/:id/knowledge-base` — upsert `knowledgeBaseId` and `dataSourceId` for the project.
  - `GET /api/projects/:id` (or project knowledge-base endpoint) — returns `knowledgeBaseId` and `dataSourceId` so the backend and worker know which KB and Data Source to use.
- **When**: Backend calls PUT after successful provisioning; GET is used when ensuring KB and enqueueing sync (e.g. for existing projects).

### GitHub webhook

- **Endpoint**: **POST /api/webhooks/docs/:projectId** (public; no JWT). Validated with `X-Hub-Signature-256` and `WEBHOOK_DOCS_SECRET`.
- **Events**: `push` — backend checks that `ref` matches `refs/heads/<docsRepoBranch>` and enqueues sync. `ping` — returns 200.
- **Ensure**: When enqueueing sync, backend may create or update the webhook on the docs repo so that push events point to this URL. Uses the same GitHub token as for sync (e.g. from user who set docsRepo).

---

## Services and env vars (summary)

| Service / component | Responsibility | Key env vars |
|--------------------|----------------|--------------|
| **ace-stack-backend** | Provisioning (S3, IAM, Bedrock KB/DataSource, S3 Vectors); build sync script; enqueue to SQS; ensure webhook; POST /api/admin/knowledge-base/sync and POST /api/webhooks/docs/:projectId | QUEUE_DOCS_SYNC_URL, AWS_REGION, API_HOST, WEBHOOK_DOCS_SECRET, DB Gateway URL, JWT |
| **ace-commands-api (docs-sync worker)** | Consume DocsSync queue; run script (clone, S3 sync, Bedrock ingestion) | QUEUE_DOCS_SYNC_URL, AWS credentials (S3, Bedrock) |
| **ace-db-gateway** | Store and return knowledgeBaseId, dataSourceId per project | — |

---

## Cleanup and deprovisioning

The backend **KnowledgeBaseProvisioningService** can **deprovision** (delete Data Source, KB, vector index, vector bucket, S3 docs bucket, IAM role and policies). This is used when a project’s KB is removed or when cleaning up test/old resources. Script **ace-infra/scripts/delete-kb-resources.sh** may be used to remove KB-related resources in bulk; see [scripts-and-automation.md](./scripts-and-automation.md). Do not run deprovision or delete scripts without confirming scope (e.g. which project or prefix).

---

## Where to find more

- **Backend API**: ace-stack-backend docs (e.g. `docs/api/knowledge-base-sync.md`, `docs/api/admin-knowledge-bases.md`).
- **Commands API worker**: ace-commands-api `docs/docs-sync-worker.md`.
- **SQS queue creation**: [scripts-and-automation.md](./scripts-and-automation.md) (create-docs-sync-queue.sh), [terraform-modules.md](./terraform-modules.md) (docs-sync-queue module).
- **Env vars**: [../environments/env-vars-and-secrets.md](../environments/env-vars-and-secrets.md) (QUEUE_DOCS_SYNC_URL, etc.).

