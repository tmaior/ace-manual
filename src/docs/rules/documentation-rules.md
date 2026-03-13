# Documentation Rules

Rules for creating, updating, and organizing documentation across the ACE system. **Everything that is done must be documented.** These rules complement [main rules](./main-rules.md) (especially "Document everything" and "Documentation structure") and apply to all repos and all contributors, including AI agents.

---

## 1. Document everything that is done

- **Every change** that affects behavior, APIs, or usage must be reflected in documentation. There are no exceptions for "small" or "internal" changes.
- **Always document**:
  - **New features** – What was added, how it works, how to use it.
  - **Bug fixes** – What was wrong and how it was fixed (when relevant for users or future maintainers).
  - **Changes in behavior or APIs** – What changed, why, and how to migrate or adapt.
  - **Architecture or infra changes** – Updated diagrams, runbooks, or architecture docs.
  - **Configuration or environment changes** – New env vars, new secrets, new deployment steps.
- Documentation is **mandatory** with each alteration. Do not leave it for a "later" PR or task.

**Agents**: After implementing or changing anything (code, config, infra), add or update the relevant docs in the **same** PR or change set. Do not commit code without the corresponding doc update.

---

## 2. When to update documentation

- **With every code change** that alters behavior, endpoints, or contracts.
- **With every new feature** – Document purpose, usage, and any new APIs or UI.
- **With every fix** that is visible to users or that changes error handling, responses, or side effects.
- **With every architectural or deployment change** – Keep architecture and environment docs in sync with reality.
- **With every critical fix** – Ensure runbooks, debugging guides, or "common errors" sections reflect the fix so the same issue is easier to diagnose next time.

**Agents**: Before considering a task done, check whether any doc (central or service-level) must be created or updated. If in doubt, document.

---

## 3. Where to document

- **Central documentation** – `ace/docs/ace-system/` (or the project’s central docs root). Use for:
  - System-wide rules, architecture, environments, and cross-service topics.
  - Place files in the correct subfolder: `rules/`, `architecture/`, `environments/`, `services/` (see [main rules](./main-rules.md)).
- **Service-level documentation** – Each microservice has a **`docs/`** directory (e.g. `ace-db-gateway/docs/`, `ace-stack-backend/docs/`). Use for:
  - Application description, setup, and behavior of that service.
  - API docs, feature descriptions, and important parts of the code.
  - Repo-specific rules and conventions.
- **Choice**: Prefer the **most relevant** place. If a change affects only one service, update that service’s `docs/`. If it affects the whole system or multiple services, update central docs and, if needed, the relevant service docs too.

**Agents**: When adding or updating docs, decide whether the change is service-specific or system-wide, then write or edit in the appropriate `docs/` (central or per-service). Do not create orphan docs in random locations.

---

## 4. Documentation structure and file naming

- **Structure**: Documentation is organized by **directories and subdirectories**. Each doc belongs in the **correct** folder (e.g. `docs/ace-system/rules/`, `docs/ace-system/architecture/`, or `ace-db-gateway/docs/api/`). Follow the existing layout; do not invent new top-level folders without aligning with the project.
- **File names**: Use **kebab-case** for all doc files (e.g. `api-endpoints.md`, `debugging-guide.md`, `local-setup.md`).
- **Entry points**: When you add, remove, or move docs, update the **index**, **README**, and **START_HERE** at that level so links and navigation stay correct (see [main rules](./main-rules.md)).

**Agents**: Use kebab-case for new filenames. Place new docs in the right directory (e.g. `api/`, `development/`, `architecture/` under a service’s `docs/` or under `ace-system/`). After any structural change, update `index.md`, `README.md`, and `START_HERE.md` as applicable.

---

## 5. Types of documentation

- **Setup guides** – How to configure and run the application or service (local, staging, production). Include prerequisites, env vars, and steps.
- **Architecture** – How the system or service is designed; components, data flow, and important decisions. Keep diagrams and descriptions up to date.
- **API documentation** – Endpoints, request/response shapes, auth, and usage examples. Update when APIs change.
- **Debugging and troubleshooting** – How to diagnose and fix common problems; common errors and their solutions.
- **Development guides** – How to contribute, run tests, and follow conventions for that repo or area.

**Agents**: When documenting a feature or change, choose the right type (setup, architecture, API, debugging, development) and add or update the corresponding doc. For APIs, include examples; for errors, include causes and resolutions.

---

## 6. Quality and examples

- **Examples** (code, config, or commands) in documentation must be **functional and tested**. Do not paste outdated or broken snippets.
- Prefer **short, clear sections** and **links** to other docs over very long single files. Break content by topic when it grows.
- Write in **English** for technical content (see [main rules](./main-rules.md)). User-facing or team-facing guides may use another language when the project agrees.

**Agents**: When adding code or command examples, verify they work (e.g. run or sanity-check). Keep explanations concise; link to related docs instead of duplicating.

---

## 7. Documentation in the same PR as the change

- Documentation updates must be part of the **same PR** (or same branch/change set) as the code or config they describe. Do not open a separate "docs only" PR later unless the project explicitly uses that workflow.
- PRs that change behavior or APIs must include the corresponding doc updates in the **PR description** (e.g. in "Documentation" or "Docs" section) and in the **diff**.

**Agents**: When implementing a feature or fix, add the doc changes in the same commits/PR. Mention in the PR body what was documented and where.

---

## 8. Service docs/ as starting point and single source of truth

- Each service’s **`docs/`** is the **starting point** for understanding that application and finding the right files to change.
- Service docs must reflect the **current** behavior of the app. With **every code update** to that service, the service **`docs/`** must be updated so it does not become outdated.
- New team members and agents should be able to rely on `docs/` to understand the app and where to work.

**Agents**: At the start of a task, read the service’s `docs/` to understand the app. After any code change in that service, update the relevant files in that `docs/` (and central docs if the change is cross-service).

---

## 9. Creating new docs and folders

- You **may create** new directories and files under **`docs/`** (central or per-service) when needed, as long as you follow:
  - The **existing structure** and naming (kebab-case, correct folder).
  - The rule to **update index, README, and START_HERE** when adding or moving docs.
  - **English** for technical content and **no invented** structure (see [main rules](./main-rules.md)).
- Do not create duplicate or conflicting docs (e.g. two "API" docs in different places for the same API without a clear reason).

**Agents**: When creating a new doc or folder, place it in the correct part of the tree, use kebab-case, and update the entry-point files. Do not create ad-hoc or duplicate documentation.

---

## Summary for AI agents

| Topic | Rule |
|-------|------|
| Scope | **Document everything** that is done (features, fixes, behavior, APIs, architecture, config). |
| When | With **every** relevant code or config change; in the **same PR**. |
| Where | Central `docs/ace-system/` for system-wide; service `docs/` for that app; choose the most relevant place. |
| Structure | Correct folder; **kebab-case** filenames; update **index**, **README**, **START_HERE**. |
| Types | Setup, architecture, API, debugging, development; examples must be **functional and tested**. |
| Quality | English for technical content; short sections; working examples; no outdated snippets. |
| Service docs | Starting point for each app; keep **current** with every change to that service. |
| New docs | Allowed within structure and rules; no duplicates or ad-hoc locations. |
