# Implementation Plan: ECU 911 RFID Access Control System

**Branch**: `001-rfid-access-control` | **Date**: 2026-09-19 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-rfid-access-control/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

Build a secure software-only RFID access management application and REST API. The MVP
will provide server-managed authentication, employee/card/door administration,
append-oriented access-event recording, and centralized filtered event review. The
implementation keeps the existing single Next.js App Router project, adds Prisma-backed
PostgreSQL persistence, validates all REST inputs at Route Handler boundaries, and uses
layered unit, integration, component, and end-to-end tests.

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: TypeScript 5.x, strict mode, Node.js runtime compatible with Next.js 16.3.5  
**Primary Dependencies**: Next.js 16.3.5 App Router, React 19.2.8, Prisma ORM, Zod, Argon2id password hashing, Vitest, Testing Library, Testcontainers, Playwright, Tailwind CSS 4  
**Storage**: PostgreSQL with Prisma migrations  
**Testing**: Vitest, Testing Library, Testcontainers PostgreSQL, Playwright, ESLint, TypeScript compiler, Next.js production build  
**Target Platform**: Responsive web application and REST API deployed on a Node.js-compatible Next.js host  
**Project Type**: Single web application  
**Performance Goals**: Valid filtered access-record queries return within 3 seconds under normal MVP operating conditions; valid authentication reaches the protected area within 5 seconds  
**Constraints**: Server-side authorization; fail-closed access decisions; UTC timestamp policy; no raw RFID identifiers in client errors or ordinary logs; physical RFID hardware excluded from MVP  
**Scale/Scope**: MVP for organization-managed users, employees, RFID cards, doors, and centralized access events; five P1 user journeys and one web/API project

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

* **PASS**: TypeScript strict mode and typed validation are required; the design forbids
  `any` and keeps domain types at module boundaries.
* **PASS**: Next.js App Router, Server Components by default, Tailwind utility classes,
  and separate domain/API logic are represented in the source structure.
* **PASS**: Authentication and role authorization are server-side, deny by default, and
  use revocable database sessions; access events are append-oriented and auditable.
* **PASS**: All REST handlers validate input and return stable HTTP/error contracts.
* **PASS**: The plan includes tests for authentication, CRUD, APIs, validation, errors,
  persistence relationships, accessibility, and primary workflows.
* **PASS**: PostgreSQL and Prisma are the sole persistence design, with restrictive
  historical relationships and migration review.
* **PASS**: No complexity exception is required; the single-project structure matches
  the repository and feature scope.

## Project Structure

### Documentation (this feature)

```text
specs/001-rfid-access-control/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
app/
├── (auth)/login/page.tsx
├── (dashboard)/layout.tsx
├── (dashboard)/page.tsx
├── (dashboard)/employees/page.tsx
├── (dashboard)/rfid-cards/page.tsx
├── (dashboard)/doors/page.tsx
├── (dashboard)/access-events/page.tsx
└── api/
  ├── auth/{login,logout,me}/route.ts
  ├── employees/route.ts
  ├── employees/[id]/route.ts
  ├── rfid-cards/route.ts
  ├── rfid-cards/[id]/route.ts
  ├── doors/route.ts
  ├── doors/[id]/route.ts
  └── access-events/route.ts

components/
├── auth/
├── employees/
├── rfid-cards/
├── doors/
└── access-events/

lib/
├── auth/               # sessions, password hashing, guards
├── db.ts               # Prisma client lifecycle
├── domain/             # access policy and domain services
├── validation/         # Zod schemas and typed parsers
└── api/                # response/error helpers

prisma/
├── schema.prisma
└── migrations/

tests/
├── unit/
├── integration/
├── contract/
├── components/
└── e2e/
```

**Structure Decision**: Use the existing root-level App Router `app/` directory and
add colocated API Route Handlers, with reusable UI in `components/`, server/domain
logic in `lib/`, Prisma schema and migrations in `prisma/`, and tests separated by
execution boundary under `tests/`. Keep access events append-only and do not add a
separate frontend/backend project for the MVP.

### Post-Design Constitution Check

* **PASS**: The data model uses explicit typed entities, strict relationships, UTC
  timestamps, restrictive event foreign keys, and deactivation instead of destructive
  deletion.
* **PASS**: The API contract applies server-managed authentication, role authorization,
  boundary validation, stable status codes, safe error responses, and no raw RFID values
  in response schemas.
* **PASS**: The single Next.js project retains Server Components by default and reserves
  client components for interactive browser behavior; business logic remains in `lib/`.
* **PASS**: Research and quickstart artifacts define Prisma migration review, layered
  tests, lint/typecheck/build gates, and accessibility/responsive verification.
* **PASS**: No new constitution violation or unjustified complexity was introduced by
  the design; the Complexity Tracking table remains intentionally empty of violations.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | The selected single-project structure and supporting domain layers satisfy the constitution without a justified violation. |
