# Development Rules

Rules for implementing and testing code changes across the ACE system. These rules apply to all development work, including work done by AI agents.

---

## 1. Feature branches from latest development

- Whenever **code changes** are required, create a **feature branch** from the **latest code** on the **`development`** branch.
- Do not branch from `main`, `staging`, or an outdated local copy. Ensure `development` is up to date (e.g. pull or fetch) before creating the feature branch.
- Use **kebab-case** for branch names (e.g. `feature/user-management`, `feature/jira-project-links`).

**Agents**: Before starting implementation, create or switch to a feature branch whose base is the current `development` branch. Do not implement on `development` directly.

---

## 2. Local testing and readiness for PR

- After implementation, the **entire system** (or the minimal set needed for the change) must be **tested locally** before the work is considered **complete** or a **PR** is opened.
- **Test mix**: Use the **code from the feature branches** you changed, together with the **code from the `development` branches** of **all other apps** that were not modified.
- **Goal**: Ensure that everything works and that services **communicate correctly** with each other (e.g. frontend ↔ backend ↔ db-gateway, bots, etc.).
- Only after confirming that the integrated scenario is correct may you treat the change as **ready for review** and open a **PR**. Use **`git push`** on the feature branch so the remote has your commits; in **ephemeral sandboxes**, push **regularly** during development (see [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md)), not only at the last second.

**Agents**: When development is complete, run the full stack locally (e.g. via docker-compose or per-service runbooks), using feature-branch code for changed services and development-branch code for unchanged ones. Do **not** treat work as **finished** or open a **PR** until this integration testing passes.

**Ephemeral sandboxes (e.g. Daytona)**: Session timeouts can **destroy local-only commits**. Follow [ai-agent-ace-workflow.md](./ai-agent-ace-workflow.md): **`git push`** the feature branch **regularly** after coherent commits so work stays visible and recoverable, while still running **integration / smoke checks** before you declare the task complete and before opening a PR.

---

## 3. Moving feature work into development

- After everything is **finished** (implementation + local testing), the **code from the feature branches** must be **moved into `development`** (typically via merge or PR into `development`, following the project’s [PR rules](./pr-rules.md) and [gitflow](./gitflow-rules.md) if documented).

**Agents**: Once local testing is successful, open a PR from the feature branch to `development` (or follow the repository’s defined process) and complete the merge. Do not leave feature work only on feature branches.

---

## 4. Service docs/ directory: read and update

- Each **microservice** in the ACE system has a **`docs/`** directory (e.g. `ace-db-gateway/docs/`, `ace-stack-backend/docs/`).
- That directory contains:
  - **Descriptions** of the application and its behavior.
  - **Documentation** of implemented features and important parts of the code.
  - **Rules** that must be followed within that repository.
- **Use it**: The service `docs/` is the **starting point** for understanding the application and for knowing **which files** to work in.
- **Keep it current**: With **every code update**, the **`docs/`** directory must be **updated** so it reflects the **current behavior** of the app. Do not leave documentation outdated.

**Agents**: At the start of a task, read the relevant service’s `docs/` to understand the app and locate the right files. After any code change, update the relevant docs (new features, changed behavior, new endpoints, etc.) in the same change set (same PR/branch).

---

## 5. Creating new docs: allowed within the rules

- The agent (or developer) **may create new directories and files** inside any service **`docs/`** (or the central docs structure) when needed.
- **Condition**: All **defined rules** must be followed (e.g. [main rules](./main-rules.md), [documentation rules](./documentation-rules.md), correct placement in the docs structure, English for technical content).
- Do not create ad-hoc or duplicate documentation that conflicts with the existing structure or conventions.

**Agents**: When adding new docs, place them in the appropriate folder (e.g. `docs/api/`, `docs/setup/`), use kebab-case filenames, write in English, and update index/README/START_HERE as required by the main and documentation rules.

---

## 6. Freedom to clone and read repositories

- You are **free to clone** any repository of the ACE system.
- You are **free to read and understand** whatever is necessary (code, configs, docs) **to achieve the goal** of the task.
- Use this to gather context, trace flows across services, and ensure your changes are consistent with the rest of the system.

**Agents**: When the task spans multiple repos or is unclear, clone or read the relevant repositories and files. Prefer reading the code and docs over guessing behavior or structure.

---

## 7. Follow existing standardizations

- **Always read** the files you need to **understand the app’s existing standardizations** (patterns, naming, structure, APIs, error handling, pagination, etc.).
- Your **updates must follow** those **existing standardizations**. Do not introduce a new style or pattern for something that already exists in the codebase.
- **Example**: If you add **pagination**, read other files in the same app that already implement pagination, and implement it with the **same format, rules, and logic**. The same applies to any other kind of feature (error handling, validation, API shape, etc.).
- Consistency reduces bugs and keeps the codebase maintainable.

**Agents**: Before implementing a feature (e.g. pagination, filtering, a new endpoint, a new UI pattern), search the repository for existing implementations of the same or similar concept and reuse the same approach. Do not invent a new pattern when one already exists.

---

## Summary for AI agents

| Topic | Rule |
|-------|------|
| Branching | Create a feature branch from latest `development`; use kebab-case. |
| Testing | Test the full system locally (feature branches + development for unchanged apps) before commit/push. |
| Merge | After success, move feature-branch code into `development` (PR/merge). |
| docs/ | Use each service’s `docs/` to understand the app and find files; update docs with every code change. |
| New docs | Allowed inside `docs/` as long as all rules (main, documentation, structure) are followed. |
| Repos | Clone and read any repo as needed to accomplish the task. |
| Standardization | Read existing code for patterns; implement new features (e.g. pagination) like existing ones. |
