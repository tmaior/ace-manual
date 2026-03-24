# Gitflow Rules

Rules for branch strategy, naming, and code flow across ACE repositories. Follow these rules together with [PR rules](./pr-rules.md) and [development rules](./development-rules.md).

---

## 1. Branch naming: kebab-case only

- **All branch names** must use **kebab-case** (lowercase, words separated by hyphens).
- **Examples**: `feature/user-management`, `bugfix/login-validation`, `hotfix/security-patch`, `feature/jira-project-links`.
- Do not use `snake_case`, `camelCase`, or spaces. Do not use uppercase.

**Agents**: When creating a branch, use only lowercase letters and hyphens. Match the prefix to the type of work: `feature/`, `bugfix/`, `hotfix/`.

---

## 2. Long-lived branches

- **development** – Integration branch for feature work. Default base for feature and bugfix branches in most repos.
- **staging** – Pre-production integration (when used). Receives merges from `development` for testing before main/production.
- **main** – Production-ready code for most repositories. PRs to `main` are typically from `staging` (or from `development` when staging is not used).
- **production** – Used in **ace-dashboard-frontend** only. Represents the live production deployment; PRs to `production` follow that repo’s release process.

Not every repository uses all of these. The exact flow depends on the repository (see section 4).

---

## 3. Short-lived branches (feature, bugfix, hotfix)

- **feature/** – New functionality or larger changes. Branch from the appropriate long-lived branch (usually **development**), then merge back via PR.
- **bugfix/** – Fixes for bugs. Same rules as feature: branch from **development** (or the branch that contains the bug), merge via PR.
- **hotfix/** – Urgent fixes that may need to go directly to staging or main. Use only when necessary; document in the PR why a hotfix was used.

**Agents**: Create feature and bugfix branches from the **latest** long-lived branch (e.g. pull `development` first, then create `feature/your-change`). Do not branch from an outdated branch or from another feature branch unless the project explicitly allows it.

---

## 4. Repository-specific flow

### Most repositories (backend, bots, db-gateway, configuration, infra, etc.)

- **Flow**: `feature` / `bugfix` → **development** → **staging** (if used) → **main**.
- **Branch from**: Create `feature/*` or `bugfix/*` from **development**.
- **Merge path**: Feature/bugfix PR into `development`; then PR from `development` to `staging` (if applicable); then PR from `staging` to `main` (or from `development` to `main` when staging is not in use).
- **Production**: PRs that deploy to production are created **from `development`** (or from the branch that your environment pipeline uses as source). Do not open production deploy PRs from `main` or ad-hoc branches unless the project defines an exception.

### ace-dashboard-frontend (exception)

- **Reason**: The frontend is edited via **Lovable**, which uses **main** as the branch for direct changes.
- **Flow**: Feature work may land on **main** first, then be promoted to **development**, then to **production**.
- **Release**: Two PRs are required for a release:
  1. **staging → main**
  2. **staging → production**
- When working in the monorepo or with non-Lovable flows, follow the same staging → main and staging → production pattern as documented for that repo.

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
| Branch names | **kebab-case** only (e.g. `feature/user-management`, `bugfix/login-validation`). |
| Long-lived branches | `development`, `staging`, `main`; `production` only in ace-dashboard-frontend. |
| Feature/bugfix | Branch from **development**; merge via PR. |
| Most repos | feature → development → (staging) → main; production deploy PRs from development. |
| ace-dashboard-frontend | Lovable uses main; release = PR staging→main + PR staging→production. |
| Commits | **Conventional Commits** (`feat`, `fix`, `docs`, etc.) in English. |
| Protected branches | No direct push; use PRs only. |
| Before PR | Update branch from base (rebase/merge), resolve conflicts. |
