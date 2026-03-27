# AI agent workflow for ACE development

This document is the **canonical playbook** for **AI agents** (and similar automation) that implement or debug ACE across repositories. It complements [main-rules.md](./main-rules.md), [development-rules.md](./development-rules.md), [gitflow-rules.md](./gitflow-rules.md), [pr-rules.md](./pr-rules.md), [security-rules.md](./security-rules.md), and [testing-rules.md](./testing-rules.md).

It exists because agents often run in **ephemeral workspaces** (e.g. **Daytona** sandboxes) where **local-only git state and files can disappear** when a session ends, and where **maintainers cannot see the filesystem** until changes are **pushed** to Git remotes.

---

## 1. Git: feature branches and push discipline

### Rules

1. **Feature branch for code changes**  
   Whenever you need to **change code** in any ACE repository, work on a **`feature/<kebab-case-purpose>`** (or `bugfix/*` / `hotfix/*` per [gitflow-rules.md](./gitflow-rules.md)) branch. Do **not** commit directly to **`development`**, **`main`**, **`staging`**, or **`production`** unless the user **explicitly** instructs otherwise.

2. **Push feature branches to the remote**  
   After **meaningful commits** on a feature branch you created for ACE work, **`git push`** (or `git push -u origin <branch>` on first push) so the branch exists on the remote. **Push regularly**—especially before pausing, before long-running commands, or when the session might time out—not only at the very end.

3. **PR base**  
   Open the PR from your feature branch **back to the branch you branched from** (almost always **`development`** for ACE service repos). Follow [pr-rules.md](./pr-rules.md). Do **not** open the PR until the user **agrees** the outcome is acceptable (see section 4).

### Rationale

- **Session loss**: Ephemeral environments (e.g. Daytona) can **close after a timeout**. Un-pushed commits and un-pushed branches exist **only locally** and can be **lost permanently**.
- **No sandbox visibility**: The user typically **cannot browse your workspace**. They see progress when it appears on **GitHub** (commits, branches, PRs). Pushing is how you **share state** and avoid duplicated or blocked work.

---

## 2. Credentials: GitHub, AWS, and project secrets

### Rules

1. **Check secrets first**  
   When an operation needs **authentication** (git push over HTTPS, `gh`, AWS CLI, ECR, deploy hooks, private package registries), **look for credentials in the environment’s secret mechanism first**: workspace/project secrets, CI variables, **GitHub Environment** secrets, Daytona or host-injected env vars, or documented secret stores. Do **not** assume they are missing without checking.

2. **Use provisioned credentials**  
   If secrets are present and scoped for the task, **use them** to complete the action. Do **not** ask the user *whether* you may use already-provisioned project credentials—**proceed**, while obeying [security-rules.md](./security-rules.md) (never print secret values, never commit them).

3. **If secrets are truly absent**  
   Only after confirming they are **not** available in the documented locations, ask the user **once** for the minimum needed configuration (or for where secrets are stored for that environment).

### Rationale

- **GitHub tokens** and **AWS credentials** for ACE automation are **intended** to live in **secrets**, not in chat. Checking secrets first matches how the platform is operated and reduces unnecessary back-and-forth.

---

## 3. Proactivity

### Rules

1. **Act when unblocked**  
   If authentication is required and **secrets/env** provide it, **run the authenticated command** (e.g. `git push`, `aws sts get-caller-identity`, API calls) instead of asking for permission to use what is already configured.

2. **Stay within safety**  
   Do **not** bypass branch protection, force-push to shared long-lived branches, or expose secrets in logs, PR descriptions, or committed files. When the user must choose between **risky** options (e.g. force-push to `main`), **stop and ask** per [main-rules.md](./main-rules.md) and team process.

### Rationale

- Reduces idle questions and speeds up delivery when the environment is **already set up** for the agent to operate.

---

## 4. Local environment, validation, definition of done, and PR

### Rules

1. **Run what you need locally**  
   Bring up **local-env** (Docker Compose) or the **minimal subset** of services needed to validate your change. Follow [environments/local.md](../environments/local.md) and [environments/local-setup/](../environments/local-setup/) in order; do not invent ports or service names.

2. **Branch checkout policy**  
   - For **every** ACE app repo you clone or refresh: use branch **`development`** unless the user **explicitly** names another branch.  
   - For **each repo where you change code**: create a **feature branch from `development`** unless the user explicitly requests a different base (e.g. hotfix flow).

3. **Definition of done includes verification**  
   Do **not** treat the task as **complete** until you have **verified** behavior that matters for the request—at minimum **smoke checks**:
   - **HTTP**: e.g. `curl` against health or relevant endpoints, expected status and response shape.  
   - **UI**: e.g. browser or **Playwright** (including MCP) flows are acceptable if you provide **evidence** (screenshots or clear step-by-step reproduction).  
   - Align with [testing-rules.md](./testing-rules.md) (simple, behavior-based checks; no obligation to add committed unit tests unless the repo already requires it).

4. **Share URLs**  
   Provide **reachable URLs** (and ports or paths if non-obvious) for the running local environment so the user can **exercise the system** themselves (e.g. dashboard, API base URL, proxy hostname documented for local-env).

5. **Pull request after user confirmation**  
   After the user **confirms** satisfaction with the implementation and tests, create the **PR** from your feature branch to **`development`** (or the branch you branched from), following [pr-rules.md](./pr-rules.md). List the PR URL in your summary.

### Rationale

- **development** as default keeps integration consistent with team gitflow.  
- **Testing before “done”** catches broken contracts and wrong env assumptions.  
- **URLs + PR** close the loop: the user can verify and merge through the normal review path.

---

## 5. Relationship to other rules

| Topic | Primary doc |
|-------|-------------|
| Branch names, protected branches, frontend dual PRs | [gitflow-rules.md](./gitflow-rules.md) |
| Feature branch from development, service `docs/` | [development-rules.md](./development-rules.md) |
| PR templates, `gh pr create` | [pr-rules.md](./pr-rules.md) |
| Secrets in code, JWT, safe errors | [security-rules.md](./security-rules.md) |
| What counts as “testing” for PRs | [testing-rules.md](./testing-rules.md) |

**Agents**: Read this file at the **start** of any **implementation** task on ACE repos from an **automated or sandboxed** environment. If anything here conflicts with a **direct user instruction** for that task, follow the user for that task and note the exception in your summary.

---

## Summary for AI agents

| Topic | Do |
|-------|-----|
| Branches | Feature (or bugfix/hotfix) branch; never commit to `development`/`main`/etc. without explicit user override. |
| Push | Push feature branch to remote **regularly**; do not rely on local-only state in ephemeral sandboxes. |
| Secrets | Discover and use project/workspace secrets for GitHub/AWS before asking the user. |
| Proactivity | Run authenticated steps when secrets exist; never leak secrets. |
| Local run | Use local-env / minimal stack; default **`development`** on all repos unless user says otherwise. |
| Done | Only after smoke or equivalent verification + share URLs. |
| PR | After **user confirms**, open PR to **`development`** (usual base) and share the link. |
