# Architecture Decision Records (ADRs)

This directory contains **Architecture Decision Records** for the ACE system. Each ADR documents an important technical or architectural decision: context, options considered, decision made, and consequences.

---

## What is an ADR?

An ADR is a short document that captures:

- **Context**: What situation or problem led to the decision.
- **Decision**: What we decided to do.
- **Consequences**: What we gain, what we give up, and what we must do going forward.

ADRs help agents and developers understand **why** the system is built a certain way and avoid reversing or contradicting past decisions without explicit discussion.

---

## How to add an ADR

1. Copy [template.md](./template.md) and save it with a new name. Use a number prefix for ordering, e.g. `0001-use-nestjs-for-backend.md`, `0002-centralize-db-access-via-gateway.md`.
2. Fill in the template: Title, Status, Context, Decision, Consequences. Optionally add Alternatives considered.
3. Add the new file to [index.md](./index.md) and [START_HERE.md](./START_HERE.md) in this directory.
4. Update the parent architecture [index.md](../index.md) and [START_HERE.md](../START_HERE.md) if the list of ADRs is summarized there (e.g. “see adr/ for N decisions”).

---

## Status values

- **Proposed**: Under discussion, not yet accepted.
- **Accepted**: Decision is in effect; current state of the system.
- **Deprecated**: No longer in effect; replaced by another ADR or by practice.
- **Superseded by [ADR-XXX](./XXXX-...md)**: Replaced by a specific later ADR.

---

## List of ADRs

See [index.md](./index.md) for a plain list of all ADR files. See [START_HERE.md](./START_HERE.md) for the same list with short descriptions.

Currently there are no numbered ADR documents yet; use the [template](./template.md) to create the first one when needed.
