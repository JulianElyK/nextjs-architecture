# Frontend Architecture Guide
**Next.js App Router · TypeScript · DDD-Inspired (Frontend Edition)**

---

## 1. Overall Goals

### What this architecture optimizes for

- **Scalable, feature-first structure**
  - Teams work by *domain/feature*, not technical layers.
- **Clear module boundaries**
  - Prevents accidental tight coupling.
- **Easy onboarding**
  - Familiar to React / Next.js developers.
  - Minimal architectural ceremony.
- **First-class App Router support**
  - Designed for Server Components, streaming, and layouts.
- **Long-term maintainability**
  - Enforced dependency rules.
  - Explicit public APIs.

### Non-goals

- Strict backend DDD / Clean Architecture
- Artificial repository or service layers
- UI treated as a secondary concern

---

## 2. High-Level Folder Structure

```txt
src/
├── app/                 # Routing & composition only
│   ├── layout.tsx
│   └── (group)/
│       ├── layout.tsx
│       └── route/
│           ├── layout.tsx
│           ├── page.tsx
│           └── @slot
│               └── page.tsx
│
├── modules/             # Feature / domain modules
│   ├── auth/
│   ├── dashboard/
│   └── billing/
│
├── core/                # Shared, domain-agnostic code
│   ├── hooks/
│   ├── components/
│   ├── utils/
│   └── config/
│
├── lib/                 # Infrastructure (non-React) & Third-party integration
│   ├── http/            # BE API endpoints integration handler
│   └── env.ts
│
└── types/               # (Optional) global shared types
└── middleware.ts
```

### Responsibility boundaries

| Folder | Responsibility |
|------|----------------|
| `app/` | Routing, layouts, providers, page composition |
| `modules/` | Business features (DDD-inspired domains) |
| `core/` | Shared, reusable, domain-agnostic UI & logic |
| `lib/` | Infrastructure & platform adapters |
| `types/` | Rare, truly global type definitions |

---

## 3. Module Design (DDD-Inspired, Frontend-Friendly)

Each **module represents a single business capability**.

### Canonical module structure

```txt
modules/auth/
├── model/        # Domain-lite logic
├── api/          # Feature-scoped API adapters
├── components/   # React components, layout, etc
├── hooks/        # React custom hooks
└── index.ts      # Explicit public API
```

---

### Layer responsibilities

#### `model/` — Domain-lite

- TypeScript types
- Schemas (e.g. Zod)
- Invariants and pure transformations

**Rules**
- No React
- No HTTP
- No browser APIs

---

#### `api/` — Feature-scoped adapters

- Talks to `lib/http`
- Maps backend data → domain models
- No React hooks

---

#### `components/` — React-only

- Components
- Screens (page-level UI)

---

#### `hooks/` — React-only

- Hooks (React Query lives here)

---

#### `index.ts` — Public API

- Defines the *only* allowed entry points
- Everything else is private

> If it is not exported here, it does not exist outside the module.

---

## 4. Dependency Rules

### Hard rules

1. `app/` may import from `modules/`, `core/`, `lib/`
2. `modules/` must never import from `app/`
3. Cross-module imports only via `index.ts`
4. No deep cross-module imports
5. No circular dependencies

---

### Internal module dependency graph

```txt
ui
 ↓
api
 ↓
model
```

---

### Cross-module dependency graph

```txt
modules/dashboard
        ↓
   modules/auth (via index.ts)
```

---

### Full system graph

```txt
            app/
             ↓
     ┌──────────────┐
     │   modules/   │
     │              │
     │ auth → model │
     │       api    │
     │       ui     │
     │              │
     │ dashboard    │
     └──────┬───────┘
            ↓
          core/
            ↓
           lib/
```

---

## 5. Cross-Module Communication Patterns

### Consuming another module

- Depend on **public behavior**, not internals
- Import only from the module root

---

### Sharing types

**Preferred**
- Export types via module `index.ts`

**Fallback**
- `types/` for rare, truly global primitives

---

### Sharing behavior

| Use case | Location |
|-------|----------|
| Cross-domain hooks | `core/hooks` |
| Pure utilities | `core/utils` |
| Feature-specific behavior | Module `ui/` |

---

### When to move something into `core/`

Move only if:
- Used by multiple modules
- Domain-agnostic
- Reusable in another product

---

### What NOT to share

- Feature UI components
- Screens
- Feature-specific hooks

**Why**
- Prevents tight coupling
- Preserves module autonomy
- Keeps refactors safe

---

## 6. React Query Integration

### Providers setup (App Router)

- `app/providers.tsx` hosts QueryClientProvider
- Imported once in `app/layout.tsx`

---

### Auth module example

- React Query hooks live in `modules/auth/ui/hooks`
- Queries scoped by feature keys

---

### Consuming auth in another module

- Import behavior via public API
- No direct API or model imports

---

### Server vs Client responsibility

| Layer | Responsibility |
|----|----------------|
| Server Components | Page data orchestration, streaming |
| Client Components | Interactivity, mutations, auth state |
| Modules | Feature execution and logic |

**Rule of thumb:** Server Components orchestrate, modules execute.

---

## 7. ESLint Enforcement

### Enforced constraints

- Prevent deep cross-module imports
- Enforce allowed dependency directions
- Detect circular dependencies

---

### Key rules

- `eslint-plugin-boundaries`
- `no-restricted-imports`
- `import/no-cycle` (optional)

Boundaries are enforced automatically — violations fail CI.

---

## 8. Operating Principles

- Modules are contracts
- `index.ts` is the law
- Share late, not early
- Optimize for refactors
- Keep `app/` thin

This architecture is designed to scale with teams, features, and time — without backend-style overengineering.

