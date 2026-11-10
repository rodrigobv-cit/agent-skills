# Full Monorepo Layout and File Responsibilities

Referenced from `vertical-slice-for-each-task/SKILL.md`.

## Full Directory Tree

```text
monorepo/
├── src/
│   ├── features/                          # Vertical Slices (1 Task = 1 Feature)
│   │   ├── <feature_name>/                # Dedicated slice folder per task
│   │   │   ├── backend/                   # Python backend for this feature
│   │   │   │   ├── router.py              # Route handlers / API controller
│   │   │   │   ├── schemas.py             # Request/response Pydantic models
│   │   │   │   ├── service.py             # Feature business logic & domain rules
│   │   │   │   ├── models.py              # Database entities (SQLAlchemy/SQLModel)
│   │   │   │   └── test_<feature_name>.py # Slice integration & unit tests
│   │   │   └── frontend/                  # TypeScript frontend for this feature
│   │   │       ├── components/            # UI components dedicated to this feature
│   │   │       ├── api/                   # API hooks & fetchers (queries/mutations)
│   │   │       ├── hooks/                 # Feature-specific state and lifecycle logic
│   │   │       └── index.ts               # Public export boundary for this feature
│   │   │
│   │   ├── users/                         # Example feature slice
│   │   │   ├── backend/
│   │   │   │   ├── router.py
│   │   │   │   ├── schemas.py
│   │   │   │   ├── service.py
│   │   │   │   ├── models.py
│   │   │   │   └── test_users.py
│   │   │   └── frontend/
│   │   │       ├── components/            # UserAvatar.tsx, UserProfileForm.tsx
│   │   │       ├── api/                   # useUserQuery.ts, userApi.ts
│   │   │       ├── hooks/
│   │   │       └── index.ts
│   │   │
│   │   └── orders/                        # Example feature slice
│   │       ├── backend/
│   │       │   ├── router.py
│   │       │   ├── schemas.py
│   │       │   ├── service.py
│   │       │   ├── models.py
│   │       │   └── test_orders.py
│   │       └── frontend/
│   │           ├── components/            # OrderList.tsx, OrderSummary.tsx
│   │           ├── api/                   # useCreateOrder.ts
│   │           ├── hooks/
│   │           └── index.ts
│   │
│   └── shared/                            # Shared NFR functionality (no business logic)
│       ├── backend/                       # Backend NFRs & infrastructure
│       │   ├── database/                  # Session manager, engine setup, migrations
│       │   ├── security/                  # Auth guards, password hashing, JWT
│       │   ├── config.py                  # Env vars, app settings (BaseSettings)
│       │   ├── exceptions.py              # Global exception handlers & RFC 7807 formats
│       │   └── utils/                     # Structured loggers, common pagination helpers
│       └── frontend/                      # Frontend NFRs & infrastructure
│           ├── components/                # Base UI design system (Button, Modal, Input)
│           ├── api/                       # Base HTTP client (Axios/Fetch wrapper with auth)
│           ├── hooks/                     # Generic utility hooks (useTheme, useDebounce)
│           └── utils/                     # Formatting utilities, global constants
│
├── contracts/                             # OpenAPI artifacts
│   └── openapi.json                       # Exported OpenAPI spec for TS generation
│
├── pyproject.toml                         # Python dependencies & tooling
├── package.json                           # TypeScript dependencies & scripts
└── tsconfig.json                          # TypeScript compiler configuration
```

## Detailed File Responsibilities

### Backend Slice (`src/features/<feature_name>/backend/`)

- **`router.py`** — Defines HTTP verbs, paths, status codes, and tags. Accepts requests, injects cross-cutting dependencies (DB session, current authenticated user), delegates execution to `service.py`, and serializes responses.
- **`schemas.py`** — Pydantic models for incoming request bodies, query/path parameters, and outgoing data contracts. Governs OpenAPI documentation generation.
- **`service.py`** — Business domain rules, calculations, validation logic, transaction handling, and repository interactions. Must not import or rely on HTTP transport details.
- **`models.py`** — SQLAlchemy or SQLModel entities, database tables, column constraints, and foreign key relations mapped to this feature.
- **`test_<feature_name>.py`** — Automated unit and integration tests asserting schema validation, service logic, and endpoint response statuses.

### Frontend Slice (`src/features/<feature_name>/frontend/`)

- **`components/`** — UI views and forms built strictly for this feature.
- **`api/`** — Data-fetching functions and hooks (e.g. TanStack Query hooks) executing requests against the backend router endpoints using auto-generated types.
- **`hooks/`** — Local UI state, custom filtering logic, or multi-step wizard state scoped solely to this feature.
- **`index.ts`** — The barrel boundary. Only components, hooks, and types explicitly exported here may be consumed by root layouts or navigation containers.

### Shared NFRs (`src/shared/`)

- **`shared/backend/`** — Database connections, connection pooling, security tokens, global exception wrappers, and environment configuration.
- **`shared/frontend/`** — Domain-agnostic design system primitives, root HTTP network clients, auth interceptors, and generic UI helpers.
