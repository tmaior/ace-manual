# Repository documentation structure: index, START_HERE, and README

Across the ACE documentation structure (in ace-manual: `src/docs/` and its subdirectories; and optionally each service's `docs/`), **every directory** that holds documentation must contain three standard files: **index.md**, **START_HERE.md**, and **README.md**. This document explains what each file is for and how to maintain them.

---

## Mandatory: create or update the three files

- **When you create a new documentation directory (folder)**: You **must** create **index.md**, **START_HERE.md**, and **README.md** inside that folder. A new folder is not complete without these three files.
- **When you add, remove, or move a document (or subdirectory)** in an existing folder: You **must** update **index.md**, **START_HERE.md**, and **README.md** in the **affected** directory so that the list, descriptions, and overview reflect the change. Do not leave the three files outdated.
- **When you edit a document** in a way that changes its purpose or role (e.g. a doc is renamed or its content no longer matches the description in START_HERE or README): Update the **START_HERE.md** and **README.md** of that directory as needed so the descriptions stay accurate.

In short: **new folder → create the three files; new/removed/moved doc (or subfolder) → edit the three files in that directory.** This is required by the [main rules](rules/main-rules.md) and applies to everyone, including AI agents.

---

## Why these three files exist everywhere

- **Consistent navigation**: Anyone opening a folder immediately finds the same entry points: an index of contents, a "start here" guide, and an overview.
- **Discoverability**: New contributors and AI agents can rely on the same structure in every doc directory and know where to look first.
- **Maintainability**: When documentation is added, removed, or reorganized, these three files are the single place to update so the structure stays accurate. See [main rules](rules/main-rules.md): *"Whenever documentation is added, removed, or reorganized, the corresponding index, README, and START_HERE files must be updated."*

---

## index.md

- **Purpose**: A **simple list** of the contents of that directory: links to every file and subdirectory in that folder. No descriptions or extra text—only the list of names and links.
- **Use**: Quick scan of "what is in this folder." Useful when you already know the structure and just need to jump to a file or subdirectory.
- **Format**: Typically a bullet list or a short table with one column (name/link). Example:
  - `[README.md](./README.md)`
  - `[some-doc.md](./some-doc.md)`
  - `[subfolder/](./subfolder/)`
- **When to update**: Whenever you add, remove, or rename a file or subdirectory in that folder. Keep the list complete and the links valid.

---

## START_HERE.md

- **Purpose**: The **entry point** for someone opening that directory for the first time. It lists the same contents as the index (files and subdirectories with links) but adds a **short description** (one or two sentences) for each item, explaining what the reader will find there.
- **Use**: "I'm in this folder—what do I read first and what is each file for?" Better formatting and descriptions than the index; helps choose what to open.
- **Format**: Clear sections (e.g. "Files", "Subdirectories"). Each entry: **bold link** followed by one or two sentences. Optional short intro at the top.
- **When to update**: Whenever you add, remove, or rename a file or subdirectory, or when the purpose of a doc changes enough that the description is no longer accurate. Keep descriptions brief and current.

---

## README.md

- **Purpose**: **Overview and usage** of that directory. It explains what the directory is for, what kind of content it holds, how to use it (e.g. "if you are new, read X then Y"), and any conventions or links to parent/related docs. It is more narrative than the index or START_HERE.
- **Use**: "What is this folder for and how do I use it?" Read when you need context, not just a list of files.
- **Format**: Short intro, optional "Purpose" or "Why this exists", optional "What you will find" (can be a table or list with short descriptions), "How to use" or "How to follow", and links to parent or related documentation. Length can vary (one short page is enough for most subdirectories).
- **When to update**: When the purpose of the directory changes, when you add or remove content and the overview or "how to use" section is affected, or when the list of documents and their roles (e.g. in a table) needs to reflect the current set of files.

---

## Summary

| File          | Role                         | Content                                      |
|---------------|------------------------------|----------------------------------------------|
| **index.md**  | Simple list of contents      | Links only (files and subdirectories).       |
| **START_HERE.md** | Entry point with context | Same items as index, plus 1–2 sentence descriptions. |
| **README.md** | Overview and how to use      | Purpose, what’s inside, how to use, links.  |

---

## Rules to follow

1. **Documentation location**: All documentation files and folders **must** be created **inside `src/docs`**. Never create docs or doc directories outside `src/docs`; `src/docs` is the single root for all repository documentation.
2. **Every documentation directory** (e.g. rules/, architecture/, environments/, infrastructure/, services/, and any new subdirectory you create under the docs root) **must** contain **index.md**, **START_HERE.md**, and **README.md**. If you create a new folder, you **must** create these three files there.
3. **Keep them in sync**: When you add, remove, or move a document (or subdirectory), you **must** update **index.md**, **START_HERE.md**, and **README.md** in the affected directory so that the list, descriptions, and overview stay correct. This is required by the [main rules](rules/main-rules.md).
4. **Same names everywhere**: Use exactly **index.md**, **START_HERE.md**, and **README.md** (lowercase, with underscore in START_HERE) so that tooling and people can rely on the same names in every folder.
5. **Scope**: Each directory’s trio applies only to **that** directory. The index and START_HERE list the contents of that folder (and its immediate subdirectories, if you list them); the README describes that folder’s role and how to use it. Subdirectories have their own index, START_HERE, and README for their own contents.

---

## For AI agents

- **Location**: Create all documentation (files and folders) **only under `src/docs`**. Never create doc directories or files outside `src/docs`.
- **New folder**: When you create a new documentation directory, you **must** create **index.md**, **START_HERE.md**, and **README.md** inside it. Do not create a doc folder without these three files.
- **New, removed, or moved doc/subdirectory**: When you add, remove, or rename a document or subdirectory, you **must** update **index.md**, **START_HERE.md**, and **README.md** in the directory where the change happened. Do not leave them outdated.
- **Summary**: New folder → create the three files. Any change to the contents of a folder (add/remove/move doc or subfolder) → edit the three files in that folder.
