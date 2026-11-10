---
name: vertical-slice-for-each-task
description: Scaffolds and enforces vertical-slice architecture across a Python backend and TypeScript frontend monorepo, giving every Feature its own isolated `features/<name>/{backend,frontend}` directory pair instead of splitting its logic across global horizontal layers. Use when `planning-and-task-breakdown` defines a Feature that needs its slice scaffolded, when deciding whether new code belongs in a feature slice or in `shared/`, or when reviewing a change for boundary leaks between features.
---

# Vertical Slice For Each Task

## Overview

Each Task from the Spec is a Feature, and each Feature gets its own vertical slice: one folder holding both its backend and frontend code, isolated from every other Feature. This lets a Feature be implemented, tested, and reviewed without touching unrelated layers or other Features' code.

## When to Use

- A Feature from `planning-and-task-breakdown` needs its implementation directories scaffolded
- Deciding whether a piece of logic belongs in a Feature slice or in `shared/`
- Reviewing a change for boundary leaks between Feature slices

**When NOT to use:** Non-functional-requirement (NFR) or infrastructure-only changes with no feature-specific business logic — those belong in `shared/`, not a new slice. Not applicable to projects without a Python backend / TypeScript frontend split.

## Core Architectural Rules

1. **Each Task from the Spec is a Feature:** business tasks and capabilities must not be split across global horizontal layers; each one is an autonomous vertical slice.
2. **Each Feature is a subfolder of `features/`:** every slice lives at `src/features/<feature_name>/`.
3. **Each Feature folder splits into `frontend/` and `backend/`:** every slice collocates its own client and server logic (`src/features/<feature_name>/backend/` and `.../frontend/`).
4. **Shared NFR functionality belongs in `shared/` only:** infrastructure and cross-cutting technical concerns (database session pooling, authentication guards, design primitives) live exclusively in `src/shared/` and must never host business rules.

## Monorepo Layout

```text
monorepo/
├── src/
│   ├── features/                # Vertical Slices (1 Task = 1 Feature)
│   │   ├── <feature_name>/
│   │   │   ├── backend/          # Python backend for this feature
│   │   │   └── frontend/         # TypeScript frontend for this feature
│   │   ├── users/                # Example feature slice
│   │   └── orders/                # Example feature slice
│   └── shared/                   # Shared NFR functionality (no business logic)
│       ├── backend/
│       └── frontend/
├── contracts/
│   └── openapi.json              # Exported OpenAPI spec for TS generation
├── pyproject.toml
├── package.json
└── tsconfig.json
```

See `references/monorepo-layout.md` for the full directory tree (including the `users`/`orders` example slices) and the per-file responsibilities inside `backend/` and `frontend/`.

## Process: Scaffolding a New Feature's Slice

1. **Scaffold the directory:** create `src/features/<feature_name>/backend/` and `src/features/<feature_name>/frontend/`.
2. **Build the backend core:**
   - Write entity tables in `backend/models.py`.
   - Define input/output DTOs in `backend/schemas.py`.
   - Implement domain rules and queries in `backend/service.py`.
   - Expose endpoints via `backend/router.py`.
   - Add slice coverage in `backend/test_<feature_name>.py`.
   - Register the new router in the application's root entrypoint.
3. **Sync types:** export the updated OpenAPI spec to `contracts/openapi.json` and regenerate the frontend's TypeScript types from it.
4. **Build the frontend slice:**
   - Implement endpoint hooks in `frontend/api/`.
   - Build UI elements in `frontend/components/`.
   - Export the public feature interface through `frontend/index.ts`.
5. **Verify isolation:** confirm no imports cross into a sibling feature's slice, and none leak business logic into `src/shared/`.

## Invariants & Boundary Rules

- **Strict isolation between features:** a feature slice must never directly import from a sibling feature slice (e.g. `features/orders/` cannot import from `features/users/`). Inter-feature coordination happens via database relations, events, or top-level aggregation.
- **One-way NFR dependency:** Features may import from `src/shared/`, but `src/shared/` must never import from `src/features/`.
- **Zero business rules in `shared/`:** anything encoding domain or business logic (pricing formulas, status checks) belongs in a feature slice, never in `shared/`.
- **Synchronized contracts:** a backend schema change requires regenerating `contracts/openapi.json` and refreshing the frontend's generated types, so API consumers stay strongly typed.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll just import this one function from another feature" | That's cross-slice coupling. Expose it via `shared/`, or coordinate through a database relation or event — or the two features aren't actually independent. |
| "This helper is small, I'll drop it in `shared/` for now" | If it encodes a business rule, it belongs in the feature slice. `shared/` is NFR/infrastructure only, regardless of size. |
| "The schema change is trivial, I'll skip regenerating the OpenAPI contract" | Silent drift between the backend schema and the frontend's generated types breaks consumers invisibly. Regenerate on every schema change. |
| "I'll start the next feature in the same files before finishing this one" | Mixing two slices' code in one place defeats isolation. Checkpoint and verify one slice before starting the next. |

## Red Flags

- A feature slice importing directly from a sibling feature slice
- Business or domain logic living in `src/shared/`
- `src/shared/` importing from `src/features/`
- A backend schema change with no corresponding `contracts/openapi.json` regeneration
- A feature slice missing its `test_<feature_name>.py`, or missing the `frontend`/`backend` split entirely

## Verification

- [ ] Both `src/features/<feature_name>/backend/` and `.../frontend/` exist for the Feature
- [ ] No feature slice imports from another feature slice
- [ ] `src/shared/` contains no business or domain logic
- [ ] `contracts/openapi.json` and the generated frontend types are refreshed after any schema change
- [ ] The slice's own tests (`test_<feature_name>.py`) pass
- [ ] Only what `frontend/index.ts` exports is consumed outside the slice

## See Also

`planning-and-task-breakdown` defines the Feature this skill scaffolds, and maps each Feature back to one Use Case in the Spec.
