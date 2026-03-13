# Testing Rules

Rules for how to test the ACE system before opening a PR or releasing. Testing is **simple and behavior-focused**: calling URLs, viewing UIs, filling forms, and checking results (including in the database). There are **no unit tests or test code committed** to the repositories; at most, **shell scripts** may be used to run checks locally, but those scripts are **not committed**.

See also: [development-rules](./development-rules.md) (local testing before commit/push), [pr-rules](./pr-rules.md) (Testing section in PR body).

---

## 1. What testing means in this project

- Testing is **manual and behavior-based**. The goal is to confirm that the system behaves correctly from the user’s or integrator’s point of view.
- **No unit tests, integration test suites, or test code** are required or committed. No Jest, Mocha, or similar test frameworks in the repo for automated test suites.
- **Lint and type check** (e.g. ESLint, TypeScript compiler) remain required where the project uses them; they are quality gates, not "tests" in the sense of this document.

**Agents**: Do not add unit tests, integration test files, or test frameworks to the codebase. When the rules say "test" or "testing", they mean the simple, manual checks described below (and optionally local shell scripts that are not committed).

---

## 2. Types of simple tests to perform

Before considering a change done (and before opening a PR), perform **simple checks** such as:

- **Calling links and URLs**: Open or request the relevant URLs (e.g. API endpoints, health checks, dashboard pages). Confirm expected status codes and that the response or page loads as expected.
- **Viewing UIs**: Open the dashboard or app UI, navigate to the affected screens, and confirm that the interface renders correctly and that data (if any) is displayed as expected.
- **Filling forms**: Submit forms (create user, create project, settings, etc.) with valid and, where useful, invalid data. Confirm success messages, redirects, or error messages, and that data is saved or rejected as expected.
- **Checking the database**: After actions that change data (e.g. create user, link project), verify in the database (or via an API that reflects the DB) that the expected rows or state exist. Use read-only queries or existing admin/API tools; do not commit one-off scripts that modify data.
- **Cross-service flows**: Where a flow involves more than one service (e.g. frontend → backend → db-gateway), run through the flow end-to-end and confirm that the full path works (e.g. form submit → API call → DB update → UI update).

**Agents**: When preparing a change or a PR, plan or perform these simple tests. Document in the PR body (Testing section) what was tested: which URLs, which UIs, which forms, and what was checked in the DB or in the response. Do not add or commit unit tests or test suites.

---

## 3. When to test

- **Before commit and push**: After implementation, run the relevant simple tests (links, UI, forms, DB checks) so that the change works in your local environment. See [development-rules](./development-rules.md) (local testing with feature branches + development for unchanged apps).
- **Before opening a PR**: Ensure the changes have been tested as above and that the PR description includes a **Testing** section with steps to reproduce and expected results (see [pr-rules](./pr-rules.md)).
- **After merge (when applicable)**: For larger or riskier changes, the same kinds of checks can be run in staging or production after deploy, following the same approach (URLs, UI, forms, DB).

**Agents**: Do not merge or approve a PR without the author having performed and documented these simple tests. Remind to add the Testing section to the PR body with concrete steps and expected results.

---

## 4. Shell scripts for local testing (optional, not committed)

- You **may** use **shell scripts** (e.g. bash) to automate simple checks locally (e.g. `curl` to endpoints, simple DB queries). Such scripts can make it easier to re-run the same checks during development.
- These scripts are **for local use only**. **Do not commit** them to the repository. Add them to `.gitignore` if they live in the repo directory, or keep them outside the repo.
- Scripts must not perform destructive or irreversible actions (e.g. dropping tables, deleting production data). Prefer read-only checks or operations on test/local data only.

**Agents**: If you generate shell scripts to run simple tests (e.g. curl, psql read-only), do not add or commit them to the repo. Tell the user they are for local execution only and should not be committed.

---

## 5. What to document in the PR (Testing section)

- In the PR body, under **Testing**, describe in English:
  - **Steps to reproduce**: e.g. "1. Start services with docker-compose. 2. Open dashboard, go to User Management. 3. Click Add User, fill form with …"
  - **What was checked**: e.g. "Confirmed user appears in list and in DB table X."
  - **Expected results**: e.g. "User is created, success message shown, record visible in DB."
- Keep it short but concrete so that a reviewer or QA can repeat the same checks if needed.

**Agents**: When helping to write a PR description, ensure the Testing section includes these elements. Do not reference unit tests or test files; reference only manual steps and, if used, local (uncommitted) scripts.

---

## Summary for AI agents

| Topic | Rule |
|-------|------|
| Scope | Simple, behavior-based tests only: URLs, UI, forms, DB checks. No unit or integration test code. |
| When | Before commit/push and before opening a PR; document in PR Testing section. |
| Shell scripts | Allowed for local automation; must not be committed. |
| PR | Testing section: steps to reproduce, what was checked, expected results. |
| Lint/type check | Still required where the project uses them; they are not "tests" under this doc. |
