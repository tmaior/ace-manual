# Pull Request (PR) Rules

Rules for creating, describing, and reviewing pull requests in the ACE system. These rules apply to all repositories unless a repository-specific section states otherwise.

---

## 1. Branch flow and base branches

- **Most repositories**: Feature work flows **ace-feat → development → staging → main**. Open PRs for each promotion (**→ development**, **development → staging**, **staging → main**) as your team uses that repo.
- **Deploy / production**: Which branch triggers production deploy is defined per repository (GitHub Actions, environments). For many services, **`main`** is the long-lived line after **staging → main**; verify workflows in the repo and ace-infra.
- **ace-dashboard-frontend**:
  - **Integration flow**: **ace-feat → development → staging → main** (same PR steps as other repos: merge to `development`, then **development → staging**, then **staging → main**).
  - **Release**: Two PRs from **staging**: **staging → main** and **staging → production**. **Production** deploy pipelines listen on the **`production`** branch; **`main`** is kept in sync for historical/tooling reasons (legacy **Lovable** read/write on `main`; Lovable is **no longer used**). For a normal release, open **both** PRs unless documented otherwise.
- **Branch names**: Always **kebab-case** (e.g. `ace-feat/user-management`, `bugfix/login-validation`).

**Agents**: Before creating a PR, confirm the current branch and the correct base (`development`, `staging`, `main`, or `production` for the second frontend promotion) per repository and target.

---

## 2. PR template selection

PR descriptions must follow the **correct template** for the branch pair. Templates are stored at the **project root** (e.g. `ace/`).

| Branch pair | Template file |
|-------------|----------------|
| ace-feat → development, bugfix → development, hotfix → development | `pr-branch-to-dev.v3.0.0.md` (or current branch-to-dev template) |
| development → staging, hotfix → staging | `pr-dev-to-staging.v2.0.0.md` (or current dev-to-staging template) |
| staging → main, hotfix → main | `pr-staging-to-main.v3.0.0.md` (or current staging-to-main template) |
| staging → production (ace-dashboard-frontend) | Use the team’s **staging → production** template if one exists; otherwise mirror the structure of `pr-staging-to-main.v3.0.0.md` and clarify deploy target in the title/body. |

- Use the template that matches the **head → base** of the PR.
- If the pair does not match any template, do not invent a format; ask or follow project-specific guidance.

**Agents**: Read the template file for the chosen branch pair and generate the PR body (e.g. `PR.md`) according to that template’s instructions.

---

## 3. PR body structure and content

The PR description must be written **in English** and include the sections required by the selected template. Typical sections:

- **New Features**: New functionality; what problem it solves and what it affects.
- **Improvements**: Changes to existing behavior (optimization, clarity, performance, structure).
- **Bug Fixes**: What was wrong and how it was fixed; how it was validated.
- **Migrations / Seeds**: New or changed migrations/seeds; backward compatibility and whether seeding is incremental.
- **Cross-Service Impact**: Other services affected (frontend, backend, bots, infra) and what was updated.
- **Security Considerations**: Secrets, auth, permissions, sensitive data handling.
- **Monitoring / Observability**: Logging, alerts, dashboards, correlation IDs.
- **Testing**: Steps to reproduce, services to run (e.g. docker-compose), expected results.
- **Dependencies**: New or updated libraries and their purpose.

**Agents**: Fill every section that the template requires. Use bullet lists and short paragraphs. For code or commands, use fenced code blocks with a **language tag** (e.g. ` ```js `, ` ```txt `).

---

## 4. Generating the PR description (PR.md)

- The PR body is generated into a file **`PR.md`** at the **project root** (repository root where the template lives).
- **Language**: Content of `PR.md` must be in **English**.
- **Diff scope**: When comparing branches to fill the description, **exclude**:
  - `node_modules/`
  - `.git/`
  - `yarn.lock`
  - `package-lock.json`
- **Overwrite**: If `PR.md` already exists, replace its contents with the new description; do not append.
- **Gitignore**: Ensure **`PR.md`** is listed in **`.gitignore`** so it is not committed.

**Agents**: When generating a PR description, run the diff between head and base, analyze the changes (optionally reading relevant files), then write or overwrite `PR.md` following the chosen template. Add `PR.md` to `.gitignore` if missing.

---

## 5. Creating the PR with GitHub CLI

- Use **`gh pr create`** with the correct base and head, and the body from `PR.md`:
  - `--base <base>` (e.g. `development`, `staging`, `main`)
  - `--head <head>` (e.g. current branch)
  - `--body-file PR.md`
  - `--title "Pull Request: <short descriptive title>"`
- Title must be concise and descriptive (e.g. derived from the main change or first section).
- If the user only asks for the description file (e.g. “just generate PR.md”), create or update `PR.md` but **do not** run `gh pr create`.

**Agents**: When the user asks to “create a PR” or “open a PR”, generate `PR.md` first, then run `gh pr create` with the appropriate flags. If the user only asks for the description, stop after writing `PR.md`.

---

## 6. Pre-PR checklist (review and quality)

Before opening or submitting a PR, ensure:

- **Self-review**: Author has reviewed their own changes.
- **Tests**: Relevant simple tests have been run (URLs, UI, forms, DB checks as per [testing-rules](./testing-rules.md)); lint and type check pass where the project uses them.
- **Documentation**: Any change in behavior or API is reflected in the relevant docs (see [main-rules](./main-rules.md)).
- **No unintended breaking changes**: Cross-service impact has been considered and documented in the PR body.
- **Local validation**: Where applicable, changes were tested locally (e.g. with docker-compose) as required by development rules.

**Agents**: When helping prepare a PR, remind or verify that these items are addressed and that the PR description documents testing steps and cross-service impact.

---

## 7. PR review process

- **Every PR** must be **reviewed and approved** before merge. Do not merge your own PR without at least one approval unless the project explicitly allows it (e.g. emergency hotfix with post-hoc review).
- **Reviewers** should verify:
  - **Build and quality**: The change builds (or the pipeline is green); lint and type check pass where applicable.
  - **Documentation**: Any change in behavior, API, or config is reflected in the relevant docs (see [documentation-rules](./documentation-rules.md)). Docs are updated in the same PR.
  - **Testing**: The PR body includes a **Testing** section with steps to reproduce, what was checked, and expected results (see [testing-rules](./testing-rules.md)). The author has run the relevant simple tests (URLs, UI, forms, DB checks).
  - **Cross-service impact**: If the change affects more than one service, the PR describes the impact and the reviewer considers whether dependent services or deployments need updates.
  - **Security**: No secrets or credentials in the diff; new or changed endpoints use validation and auth (JWT) as per [security-rules](./security-rules.md) and project patterns.
- **Approval** means the reviewer has checked these points (or explicitly accepted exceptions) and approves the merge. Approval can be conditional on minor fixes; request changes when the PR does not meet the above.

**Agents**: When asked to review or to describe the review process, refer to this section. Do not approve a PR without confirming that docs are updated, testing is documented, and the change aligns with security and standardization rules.

---

## 8. Repository-specific notes

- **ace-dashboard-frontend**: For releases, both **staging → main** and **staging → production** PRs are required (`production` is what the live deploy pipeline watches; `main` is kept aligned—see [gitflow rules](./gitflow-rules.md)).
- **ace-infra**: Follow the same PR rules; additionally, **terraform fmt** and **terraform validate** must pass before merge.

---

## Summary for AI agents

| Topic | Rule |
|-------|------|
| Base branch | Most repos: ace-feat→development→staging→main. Frontend release: also **staging→production** (plus staging→main). |
| Branch names | kebab-case only. |
| Template | Choose template by branch pair (branch-to-dev, dev-to-staging, staging-to-main). |
| PR body | English; all template sections filled; code blocks with language. |
| PR.md | At project root; overwrite; in .gitignore; exclude node_modules, .git, lock files from diff. |
| gh pr create | Use --body-file PR.md and a short, descriptive title. |
| Before PR | Self-review, simple tests run and documented, docs updated, no unintended breaking changes. |
| Review | Every PR needs approval; reviewer checks build, docs, testing section, cross-service impact, security. |
