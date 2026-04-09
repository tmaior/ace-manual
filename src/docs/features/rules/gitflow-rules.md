# Gitflow Rules

Rules for branch strategy, naming, and code flow across ACE repositories. Follow these rules together with [PR rules](./pr-rules.md) and [development rules](./development-rules.md).

---

## 1. Branch naming: kebab-case only

- **All branch names** must use **kebab-case** (lowercase, words separated by hyphens).
- **Examples**: `ace-feat/user-management`, `bugfix/login-validation`, `hotfix/security-patch`, `ace-feat/jira-project-links`.
- Do not use `snake_case`, `camelCase`, or spaces. Do not use uppercase.

**Agents**: When creating a branch, use only lowercase letters and hyphens. Match the prefix to the type of work: `ace-feat/`, `bugfix/`, `hotfix/`.

---

## 2. Long-lived branches

- **development** – Integration branch for feature work. Default base for feature and bugfix branches in most repos.
- **staging** – Pre-production integration (when used). Receives merges from `development` for testing before main/production.
- **main** – Production-ready code for most repositories. PRs to `main` are typically from `staging` (or from `development` when staging is not used).
- **production** – Used in **ace-dashboard-frontend** only. Represents the live production deployment; PRs to `production` follow that repo’s release process.

Not every repository uses all of these. The exact flow depends on the repository (see section 4).

---

## 3. Short-lived branches (ace-feat, bugfix, hotfix)

- **ace-feat/** – New functionality or larger changes. Branch from the appropriate long-lived branch (usually **development**), then merge back via PR.
- **bugfix/** – Fixes for bugs. Same rules as ace-feat: branch from **development** (or the branch that contains the bug), merge via PR.
- **hotfix/** – Urgent fixes that may need to go directly to staging or main. Use only when necessary; document in the PR why a hotfix was used.

**Agents**: Create ace-feat and bugfix branches from the **latest** long-lived branch (e.g. pull `development` first, then create `ace-feat/your-change`). Do not branch from an outdated branch or from another ace-feat branch unless the project explicitly allows it.

---

## 4. Repository-specific flow

### Most repositories (backend, bots, db-gateway, configuration, infra, etc.)

- **Flow**: `ace-feat` / `bugfix` → **development** → **staging** → **main**.
- **Branch from**: Create `ace-feat/*` or `bugfix/*` from **development**.
- **Merge path**: Feature/bugfix PR into **development**; then PR **development → staging**; then PR **staging → main**.
- **Deploy**: Runtime deploy for each service follows that repo’s CI/CD (GitHub Actions, branch filters, environments). Often production-like environments track **`main`** after **staging → main**; confirm in the repo’s workflows and [ace-infra](../infrastructure/ace-infra-repository.md).

### ace-dashboard-frontend

- **Day-to-day flow**: `ace-feat` / `bugfix` → **development** → **staging** → **main**, same PR chain as other repos (**development → staging**, then **staging → main**).
- **Release promotions from staging**: After validation on **staging**, two separate PRs are required:
  1. **staging → main**
  2. **staging → production**
- **Why two PRs from staging**: **CI/CD that deploys the live customer-facing site listens on the `production` branch**, while **`main` historically existed for Lovable**, which was configured to read and write the **main** branch directly. **Lovable is no longer used**, but the split remains: one promotion updates **`main`** (e.g. continuity with tooling and history) and the other updates **`production`** (what the production pipeline deploys). Always open **both** PRs for a coordinated frontend release unless the team explicitly documents a different process.

**Agents**: Before creating or merging a PR, confirm which repository you are in and apply the correct flow (most repos vs ace-dashboard-frontend). Use the PR template that matches the branch pair (see [PR rules](./pr-rules.md)).

---

## 5. Commits: Conventional Commits

- **Commit messages** must follow **Conventional Commits**.
- **Format**: `<type>(<scope>): <description>`.
- **Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, etc. Use lowercase.
- **Examples**:
  - `feat(users): add pagination to user list`
  - `fix(auth): correct JWT validation for expired tokens`
  - `docs(api): update endpoints for project links`
- Optional: add a body and/or footer (e.g. `BREAKING CHANGE:`, `Refs #123`).

**Agents**: When committing, use a conventional type and a clear, short description in English. Do not use vague messages like "update" or "fix" without scope/description.

---

## 6. No direct push to protected branches

- **development**, **staging**, **main**, and **production** (where it exists) are **protected**. Code reaches them only via **pull requests** (and, where allowed, via merge from the previous branch in the flow).
- Do not force-push to long-lived branches. Do not bypass PRs for the sake of speed unless the project explicitly allows it (e.g. emergency hotfix with post-hoc review).

**Agents**: Always open a PR to merge into development, staging, main, or production. Do not suggest or perform direct pushes to these branches.

---

## 7. Keep branches up to date

- Before opening a PR, **rebase or merge** the target branch (e.g. `development`) into your feature branch so that your branch is up to date and conflicts are resolved early.
- Prefer a linear history where the project allows it (rebase before merge). If the project uses merge commits, follow that convention.

**Agents**: When preparing a PR, run `git fetch` and update your branch from the base branch (e.g. `git rebase origin/development` or `git merge origin/development`), then push. Resolve conflicts before requesting review.

---

## Summary for AI agents

| Topic | Rule |
|-------|------|
| Branch names | **kebab-case** only (e.g. `ace-feat/user-management`, `bugfix/login-validation`). |
| Long-lived branches | `development`, `staging`, `main`; `production` only in ace-dashboard-frontend. |
| Feature/bugfix | Branch from **development**; merge via PR. |
| Most repos | ace-feat → development → staging → main; deploy source per repo CI/CD (often main after staging→main). |
| ace-dashboard-frontend | ace-feat → development → staging → main; release = **two PRs**: staging→main **and** staging→production (production pipeline watches `production`; `main` retained from legacy Lovable setup). |
| Commits | **Conventional Commits** (`feat`, `fix`, `docs`, etc.) in English. |
| Protected branches | No direct push; use PRs only. |
| Before PR | Update branch from base (rebase/merge), resolve conflicts. |
