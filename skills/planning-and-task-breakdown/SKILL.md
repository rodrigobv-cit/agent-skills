---
name: planning-and-task-breakdown
description: Breaks work into ordered Features and Tasks. Use when you have a spec or clear requirements and need to break work into implementable units. Use when a task feels too large to start, when you need to estimate scope, or when parallel work is possible.
---

# Planning and Task Breakdown

## Overview

A Spec is made of several Use Cases. Each Use Case becomes one Feature, and each Feature is scaffolded as one vertical slice (backend + frontend) using the `vertical-slice-for-each-task` skill. Decompose each Feature into small, verifiable Tasks with explicit acceptance criteria. Good task breakdown is the difference between an agent that completes work reliably and one that produces a tangled mess. Every task should be small enough to implement, test, and verify in a single focused session.

"Use Case" here is the same concept the `spec-driven-development` skill calls a "capability" in its capability map — one Use Case/capability in the Spec, one Feature here.

## When to Use

- You have a Spec and need to break it into Features and their implementable units (Tasks)
- A Feature is a set of Tasks, and every Feature must trace back to a Use Case in the Spec
- A Feature feels too large or vague to start
- Work needs to be broken into Tasks and, if possible, parallelized across multiple agents or sessions
- You need to communicate scope to a human
- The implementation order isn't obvious

**When NOT to use:** Single-file changes with obvious scope, or when the spec already contains well-defined tasks.

## The Planning Process

### Step 1: Enter Plan Mode

Before writing any code, operate in read-only mode:

- Read the Spec and relevant codebase sections
- Identify existing patterns and conventions
- Map dependencies between Features and between Tasks within a Feature
- Note risks and unknowns

**Do NOT write code during planning.** The output is a plan document saved to `tasks/plan.md` and a task list recorded in the task list target (see Output Files; default `tasks/todo.md`), not implementation.

### Step 2: Identify the Dependency Graph

Map what depends on what:

```
Database schema
    │
    ├── API models/types
    │       │
    │       ├── API endpoints
    │       │       │
    │       │       └── Frontend API client
    │       │               │
    │       │               └── UI components
    │       │
    │       └── Validation logic
    │
    └── Seed data / migrations
```

Implementation order follows the dependency graph bottom-up: build foundations first.

### Step 3: Slice Vertically

Instead of building all the database, then all the API, then all the UI — build one complete Feature path at a time:

**Bad (horizontal slicing):**
```
Task 1: Build entire database schema
Task 2: Build all API endpoints
Task 3: Build all UI components
Task 4: Connect everything
```

**Good (vertical slicing):**
```
Feature 1: User can create an account (schema + API + UI for registration)
  Task 1: Build the GUI for account registration
  Task 2: Build the API that connects frontend and backend for registration
  Task 3: Build the backend logic to create an account

Feature 2: User can log in (auth schema + API + UI for login)
  Task 1: Build the GUI for login
  Task 2: Build the API that connects frontend and backend for login
  Task 3: Build the backend logic to authenticate a login
```

Each vertical slice corresponds to one Feature and delivers working, testable functionality. Every Feature must be broken into Tasks. Scaffold each Feature's slice using the `vertical-slice-for-each-task` skill, which defines the on-disk layout (one `features/<name>/{backend,frontend}` pair per Feature) and the boundary rules between slices.

### Step 4: Write Tasks

Each task follows this structure, whether it lands in the markdown task list or as an item in an external tracker (see Output Files):

```markdown
## Task [N]: [Short descriptive title]

**Description:** One sentence explaining what this task accomplishes.

**Acceptance criteria:**
- [ ] [Specific, testable condition]
- [ ] [Specific, testable condition]

**Verification:**
- [ ] Tests pass: [the repository's focused-test command]
- [ ] Build succeeds: [the repository's build command]
- [ ] Manual check: [description of what to verify]

**Dependencies:** [Task numbers this depends on, or "None"]

**Files likely touched:**
- `src/path/to/file.ts`
- `tests/path/to/test.ts`

**Estimated scope:** [Small: 1-2 files | Medium: 3-5 files | Large: 5+ files]
```

### Step 5: Order and Checkpoint

Arrange Features and Tasks so that:

1. Dependencies are satisfied (build foundation first)
2. Each Task leaves the system in a working state
3. Verification checkpoints occur after every 2-3 Tasks
4. High-risk Tasks are early (fail fast)
5. 1 Feature = 1 Use Case from the Spec (state the Use Case number explicitly)
6. 1 Feature = 1 vertical slice (one backend and one frontend)
7. 1 Feature is composed of N Tasks; each Task can be validated separately

Add explicit checkpoints to the task list target:

```markdown
## Checkpoint: After Tasks 1-3
- [ ] All tests pass
- [ ] Application builds without errors
- [ ] Core user flow works end-to-end
- [ ] Review with human before proceeding
```

## Feature Sizing Guidelines

| Size | Files | Scope | Example |
|------|-------|-------|---------|
| **XS** | 1 | Single function or config change | Add a validation rule |
| **S** | 1-2 | One component or endpoint | Add a new API endpoint |
| **M** | 3-5 | One feature slice | User registration flow |
| **L** | 5-8 | Multi-component feature | Search with filtering and pagination |
| **XL** | 8+ | **Too large — split into another Feature** | — |

If a Feature is L or larger, split it into another Feature rather than growing it in place. Every Feature is composed of Tasks, and Tasks should always be sized S or XS — an agent performs best on S/XS Tasks.

**When to split a Feature, or break a Task down further:**
- It would take more than one focused session (roughly 2+ hours of agent work)
- You cannot describe the acceptance criteria in 3 or fewer bullet points
- It touches two or more independent subsystems (e.g., auth and billing)
- You find yourself writing "and" in the Feature or Task title (a sign it is two of them)

## Output Files

- **Plan document:** Save the implementation plan to `tasks/plan.md`. This is always a markdown file — design decisions, risks, and open questions don't map cleanly onto individual tracker issues.
- **Task list:** Record each Feature's Tasks in the **task list target** (defined below).

Create the `tasks/` directory if it does not exist.

**Never overwrite an incomplete plan.** Before writing `tasks/plan.md` or `tasks/todo.md`, check whether they already exist and still contain unchecked tasks:

- Same work being replanned (the user asked to revise or extend this plan) → update the existing files in place.
- Different work → **stop and ask.** The unchecked tasks may be mid-build in another session. Do not delete, overwrite, or rename the existing files on your own; present the conflict and let the user decide (finish the old plan first, explicitly discard it, or tell you where the new plan should go).

The same rule applies to an external task list target: never bulk-close or delete another plan's open tracker items to make room for new ones.

### Task List Target

The task list target is where Features, Tasks, and checkpoints are recorded. It is defined once, here; every other reference in this skill defers to it.

- **Default: a single checklist-style markdown file at `tasks/todo.md`, organized into one section per Feature, each containing that Feature's Tasks.** This is the convention the `/build` command and other downstream tooling expect. Use it unless the project says otherwise.
- **External tracker:** if the project's agent rules (`CLAUDE.md`, `AGENTS.md`, etc.) or the user designate an issue tracker (e.g. GitHub Issues, Jira, Linear, `bd`/beads), create one tracker item per Task instead of writing `tasks/todo.md`, grouped under a parent/epic tracker item per Feature where the tracker supports it. Map the Step 4 structure onto the tracker's fields: acceptance criteria and verification steps in the item body, dependencies via the tracker's linking mechanism (`bd dep add`, "blocked by", etc.). Record Step 5 checkpoints as tracker items too, or as a checklist in the plan document if the tracker has no natural equivalent.

When using an external tracker, note it in `tasks/plan.md` (e.g. "Tasks tracked in Linear project FOO") so downstream steps and future sessions know where to look, and keep the plan document's Task List section as an ordered index of tracker item IDs or links rather than a duplicate checklist.

## Plan Document Template

```markdown
# Implementation Plan: [Project Name]

## Overview
[One paragraph summary of what we're building]

## Architecture Decisions
- [Key decision 1 and rationale]
- [Key decision 2 and rationale]

## Features List and Task List

### Feature 1: [Name] (Use Case [N])
- [ ] Task 1: ...
- [ ] Task 2: ...
- [ ] Task 3: ...

### Checkpoint: Feature 1 complete
- [ ] All tests pass, builds clean
- [ ] Feature's end-to-end flow works

### Feature 2: [Name] (Use Case [N])
- [ ] Task 1: ...
- [ ] Task 2: ...

### Checkpoint: Feature 2 complete
- [ ] All tests pass, builds clean
- [ ] Feature's end-to-end flow works

### Checkpoint: Complete
- [ ] All acceptance criteria met across every Feature
- [ ] Ready for review

## Risks and Mitigations
| Risk | Impact | Mitigation |
|------|--------|------------|
| [Risk] | [High/Med/Low] | [Strategy] |

## Open Questions
- [Question needing human input]
```

When tasks live in an external tracker, keep the Features List and Task List section above as an ordered index of tracker item IDs or links instead of a duplicate checklist.

## Parallelization Opportunities

When multiple agents or sessions are available:

- **Safe to parallelize:** Independent Features (separate vertical slices with no shared API contract) are the primary parallelization unit; also tests for already-implemented features, and documentation
- **Must be sequential:** Database migrations, shared state changes, dependency chains
- **Needs coordination:** Features that share an API contract (define the contract first, then parallelize)

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll figure it out as I go" | That's how you end up with a tangled mess and rework. 10 minutes of planning saves hours. |
| "The tasks are obvious" | Write them down anyway. Explicit tasks surface hidden dependencies and forgotten edge cases. |
| "Planning is overhead" | Planning is the task. Implementation without a plan is just typing. |
| "I can hold it all in my head" | Context windows are finite. Written plans survive session boundaries and compaction. |
| "The old `tasks/plan.md` is stale, I'll just replace it" | Unchecked tasks may be mid-build in another session. Overwriting them destroys work state that exists nowhere else. Stop and ask. |

## Red Flags

- Starting implementation without a written task list
- Overwriting a `tasks/plan.md` or `tasks/todo.md` that still has unchecked tasks for different work, without asking
- Writing `tasks/todo.md` when the project has designated an external tracker (or scattering tasks across both)
- Tasks that say "implement the feature" without acceptance criteria
- No verification steps in the plan
- All Features are XL-sized
- A Feature with no corresponding Use Case in the Spec
- A Task sized larger than S/XS
- No checkpoints between Features
- Dependency order isn't considered

## Verification

Before starting implementation, confirm:

- [ ] Every Feature maps to exactly one Use Case in the Spec
- [ ] Each Feature's slice follows the `vertical-slice-for-each-task` skill's layout
- [ ] Every Task has acceptance criteria
- [ ] Every Task has a verification step
- [ ] Task dependencies are identified and ordered correctly
- [ ] Tasks are recorded in the task list target (default `tasks/todo.md`)
- [ ] No pre-existing incomplete plan was overwritten without explicit user confirmation
- [ ] No Task exceeds S size (≤2 files); larger work is split into another Feature, not a bigger Task
- [ ] Checkpoints exist between Features
- [ ] The human has reviewed and approved the plan

## See Also

Acceptance criteria are per-task and answer "did we build the right thing?". They sit on top of the project-wide Definition of Done, the standing bar every task clears before it counts as done. See `../../references/definition-of-done.md`.

Use the `vertical-slice-for-each-task` skill to scaffold and verify each Feature's on-disk slice. Use Cases/Features originate from the `spec-driven-development` skill's capability map.
