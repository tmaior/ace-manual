# Main Rules

These rules **must always be followed** by anyone (human or AI agent) working on the ACE system. They take precedence over repository-specific conventions when in conflict.

---

## 1. Language: English only

- **Code**: All source code, including comments, must be in English.
- **Naming**: Function names, variables, types, constants, file names (when descriptive), and identifiers must be in English.
- **Documentation**: All technical documentation must be written in English (including content in `docs/` and in each service's `docs/` folder).
- **Exception**: User-facing communication (e.g. UI copy, messages to end users) may use another language when required by the product (e.g. Portuguese).

**Agents**: When generating or modifying code or docs, output in English unless the task explicitly asks for user-facing text in another language.

---

## 2. Document everything that is implemented or changed

- Every **new feature**, **change in behavior**, or **fix** must be reflected in documentation.
- Documentation may live in:
  - Central documentation repositories or structure (e.g. `ace/docs/ace-system/`), or
  - The `docs/` folder of the microservice (or repo) that was modified.
- Prefer documenting in the most relevant place; if it affects multiple services or the whole system, update both central and service-level docs as needed.

**Agents**: After implementing or altering functionality, add or update the relevant docs in the same scope of work (same PR/task). Do not leave documentation for "later."

---

## 3. Documentation structure: use the right place

- Documentation is organized by **directories and subdirectories** that group related content.
- Each doc must be placed in the **correct** part of that structure (e.g. `rules/`, `architecture/`, `environments/`, `services/` under `ace-system/`, or the appropriate subfolder inside a service's `docs/`).
- Do not create ad-hoc locations; follow the existing structure. If a new category is needed, align with the project's documentation conventions first.

**Agents**: Before creating or moving a doc, check the existing layout (e.g. `docs/ace-system/`, `docs/ace-system/rules/`, and each service's `docs/`) and place the file in the appropriate folder.

---

## 4. Keep index, README and START_HERE in sync

- Whenever documentation is **added, removed, or reorganized**, the corresponding **index**, **README**, and **START_HERE** files must be updated.
- These files act as entry points and maps; they must reflect the current structure and links.
- Apply this at the level where the change happened (e.g. under `docs/ace-system/` or under a service's `docs/`).

**Agents**: When creating, renaming, or deleting docs, update the relevant `index.md`, `README.md`, and `START_HERE.md` so that links and listed sections remain accurate.

---

## 5. Respect existing standardizations

- Existing **standardizations** (naming, patterns, APIs, folder layout, commit format, etc.) must be respected unless explicitly changing them via a dedicated decision or PR.
- Do not introduce new conventions in isolation; follow what is already documented (e.g. in `rules/`, in `.cursor/rules/`, or in service `docs/`).

**Agents**: Before introducing a new pattern or style, check the rules and docs (including standardization) and align with them. If a rule is missing or unclear, prefer consistency with the rest of the codebase.

---

## 6. Always follow the rules and read the relevant docs

- These main rules and all other project rules (PRs, development, infrastructure, documentation, standardization) are **mandatory**.
- Before performing an action (e.g. opening a PR, changing infra, adding a feature), **read the documentation that applies to that action** (e.g. PR rules, development rules, service-specific docs).

**Agents**: At the start of a task, identify which rules and docs apply (main rules, PR, development, infrastructure, documentation, standardization, and any service-specific docs). Use them to guide implementation and avoid violations.

---

## 7. Nothing must be invented

- **Do not invent** behavior, conventions, or implementation details. If the agent does not know **what** to do or **how** to do it, it must **read the documentation and the application code** until it understands what should be done.
- Only if it is **still unclear** after reading the relevant docs and code may the agent **ask the user** for clarification.
- Guessing or making up patterns, APIs, or workflows is not allowed. Prefer reading first, then asking, over inventing.

**Agents**: When something is ambiguous or unknown, search and read the project docs (e.g. `docs/ace-system/`, service `docs/`, `.cursor/rules/`) and the relevant application code. If after that you still do not know what or how to do it, ask the user. Do not invent solutions, naming, or behavior.

---

## Summary for AI agents

| Rule | Short reminder |
|------|----------------|
| 1    | Use English for code, names, and technical docs. |
| 2    | Document every implementation or change. |
| 3    | Put docs in the correct directory structure. |
| 4    | Update index, README, and START_HERE when docs change. |
| 5    | Respect existing standardizations. |
| 6    | Always read and follow the rules and relevant docs before acting. |
| 7    | Do not invent; read docs and code until you understand; if still unclear, ask the user. |
