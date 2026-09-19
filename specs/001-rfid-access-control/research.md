# Research: ECU 911 RFID Access Control System

## Decision 1: Database-backed sessions with opaque HttpOnly cookies

**Decision**: Use a `Session` table in PostgreSQL. Store only a hash of a cryptographically random session token, return the opaque token in an `HttpOnly` cookie, and resolve the user server-side for every protected page and Route Handler. Use Argon2id for password hashes, with `SameSite=Lax`, `Secure` in production, and `Path=/` cookie settings.

**Rationale**: Immediate logout, session revocation, disabled-user enforcement, and role changes are important for an access-control application. Database sessions provide these controls more directly than stateless JWTs. Server-side authorization also satisfies the constitution's fail-closed requirement.

**Alternatives considered**:
- Auth.js or Better Auth: viable and safer than hand-rolled authentication for a larger product, but introduces more framework configuration than the MVP requires. Revisit if external identity providers, MFA, or account recovery become requirements.
- JWT cookie sessions: rejected for the MVP because revocation and role changes are less direct.
- External identity provider: deferred because the specification assumes organization-managed username/email and password authentication for the first release.

**Implications**: Add `User` and `Session` models, a shared `requireUser`/`requireRole` server helper, login/logout/me Route Handlers, generic login errors, login rate limiting, and tests for expired, revoked, inactive, and unauthorized sessions.

## Decision 2: Prisma models preserve event history and assignment context

**Decision**: Use PostgreSQL with Prisma models for `User`, `Session`, `Employee`, `RfidCard`, `RfidCardAssignment`, `Door`, and append-oriented `AccessEvent`. Store RFID identifiers as a deterministic server-side digest (with a safe display suffix if needed), not as values returned to clients or logs. Deactivate employees, cards, and doors rather than deleting records referenced by events.

**Rationale**: A mutable current card-to-employee relation alone cannot explain card ownership after reassignment. An assignment-history model or immutable event snapshot is needed for trustworthy audit review. A separate assignment model provides normalized history and allows one active assignment per card. Access events retain employee, card, door, decision, reason, occurred time, and recording time.

**Alternatives considered**:
- Store only `employeeId` on `RfidCard` and `AccessEvent`: simpler, but card reassignment can make historical ownership ambiguous.
- Store raw RFID identifiers: rejected because identifiers are sensitive credentials and the constitution prohibits exposing them in ordinary logs or client errors.
- Hard delete domain records: rejected because it breaks referential integrity and audit history.

**Implications**: Use restrictive foreign-key behavior for event relations, partial uniqueness for one active card assignment, UTC `timestamptz(3)` values, indexes supporting employee/card/door/date filters, and transactions around authorization plus event insertion.

## Decision 3: REST Route Handlers with boundary validation and stable errors

**Decision**: Implement REST endpoints as Next.js App Router Route Handlers under `app/api/`. Validate route params, query params, headers, and bodies at the boundary with typed schemas. Use consistent JSON error responses and status codes: `401` unauthenticated, `403` unauthorized, `400` malformed, `422` semantically invalid, `409` uniqueness/conflict, `404` missing resource, `201` persisted creation, and `200` successful reads/updates.

**Rationale**: The feature is explicitly a REST API and the constitution requires server-side authorization, validation, clear errors, and no stack-trace leakage. A single validation boundary reduces duplicated checks across UI and API clients.

**Alternatives considered**:
- Server Actions as the primary contract: useful for UI mutations but does not satisfy the required REST API surface.
- GraphQL: not requested and adds schema/runtime complexity for a focused MVP.
- Returning raw database errors: rejected because it leaks implementation details and produces unstable client behavior.

**Implications**: Define shared domain services separate from page components, keep access events append-only without PATCH/DELETE handlers, combine all supplied list filters with `AND`, validate half-open UTC date ranges, and return `201` only after persistence succeeds.

## Decision 4: Layered testing with Vitest, Testing Library, Testcontainers, and Playwright

**Decision**: Use Vitest for unit and Route Handler tests, Testing Library for interactive client components, Testcontainers with PostgreSQL for Prisma integration tests, and Playwright for end-to-end authentication and primary workflows.

**Rationale**: The constitution requires authentication, CRUD, API, validation, and error-case coverage. Unit tests alone cannot validate PostgreSQL constraints or session behavior, while browser tests alone are too slow and opaque for domain rules. The layered approach keeps feedback targeted and protects the real data relationships.

**Alternatives considered**:
- Jest: viable, but Vitest is a straightforward fit for modern TypeScript/ESM tooling.
- Mocked Prisma only: rejected for relationship, migration, uniqueness, and referential-integrity behavior.
- Cypress: viable, but Playwright provides strong multi-browser and mobile viewport workflow coverage.

**Implications**: Add test scripts and configuration during implementation. Cover authorization boundaries, duplicate conflicts, inactive records, invalid filters, persistence failures, append-only events, keyboard access, and responsive primary workflows.

## Decision 5: Single Next.js project structure

**Decision**: Keep one web project with App Router pages under `app/`, API Route Handlers under `app/api/`, Prisma schema under `prisma/`, domain logic under `lib/`, and tests under `tests/` grouped by unit, integration, contract, and e2e concerns.

**Rationale**: The repository is a single Next.js application, and separating frontend/backend projects would add deployment and coordination complexity without a requirement for independent services.

**Alternatives considered**:
- Separate `frontend/` and `backend/` applications: rejected for the MVP because Next.js Route Handlers already provide the required API boundary.
- A generic `src/` migration: possible, but unnecessary churn for the existing App Router scaffold; retain the current root-level `app/` convention.
