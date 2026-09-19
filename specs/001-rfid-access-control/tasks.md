# Tasks: ECU 911 RFID Access Control System

**Input**: Design documents from `/specs/001-rfid-access-control/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/openapi.yaml, quickstart.md
**Tests**: Included because the specification and constitution require authentication, CRUD, API, validation, error-case, accessibility, and end-to-end verification.
**Organization**: Tasks are grouped by user story so each increment can be implemented and tested independently after the shared foundation.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Install the required runtime, persistence, validation, and testing tooling without changing feature behavior.

- [ ] T001 Install Prisma, @prisma/client, Zod, and Argon2 dependencies in `package.json`
- [ ] T002 [P] Install Vitest, React Testing Library, Playwright, Testcontainers, and supporting test dependencies in `package.json`
- [ ] T003 [P] Add test, test:watch, test:e2e, and test:all scripts to `package.json`
- [ ] T004 [P] Create Vitest configuration in `vitest.config.ts` with TypeScript path aliases and separate Node/jsdom environments
- [ ] T005 [P] Create Playwright configuration in `playwright.config.ts` with the local Next.js web server and desktop/mobile projects
- [ ] T006 [P] Add test and environment-file exclusions to `.gitignore` for generated reports, local databases, and secrets
- [ ] T007 [P] Create the planned directory structure under `components/`, `lib/`, `prisma/`, and `tests/`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Build the shared database, authentication, validation, API response, and testing infrastructure required by every user story.

**⚠️ CRITICAL**: No user story implementation can begin until this phase is complete.

- [ ] T008 Create the PostgreSQL Prisma schema for User, Session, Employee, RfidCard, RfidCardAssignment, Door, and AccessEvent in `prisma/schema.prisma`
- [ ] T009 Add restrictive event foreign keys, unique identifiers, active-assignment constraints, UTC timestamps, and filtering indexes in `prisma/schema.prisma`
- [ ] T010 Generate and review the initial Prisma migration in `prisma/migrations/`
- [ ] T011 [P] Implement the singleton Prisma client lifecycle in `lib/db.ts`
- [ ] T012 [P] Implement environment parsing and required secret validation in `lib/config.ts`
- [ ] T013 [P] Implement password hashing and verification with Argon2id in `lib/auth/password.ts`
- [ ] T014 [P] Implement opaque token generation, hashing, cookie options, and session expiration helpers in `lib/auth/session.ts`
- [ ] T015 Implement database session creation, lookup, revocation, and inactive-user rejection in `lib/auth/session-service.ts`
- [ ] T016 Implement server-side `requireUser` and `requireRole` authorization guards in `lib/auth/guards.ts`
- [ ] T017 [P] Define shared Zod schemas and typed parsers for UUIDs, pagination, UTC timestamps, date ranges, and common fields in `lib/validation/common.ts`
- [ ] T018 [P] Define stable JSON success and error response helpers with status-code mapping in `lib/api/responses.ts`
- [ ] T019 [P] Implement structured safe logging that excludes passwords, session tokens, raw RFID identifiers, and unnecessary personal data in `lib/api/logging.ts`
- [ ] T020 [P] Add the initial Vitest setup, Testing Library matchers, and test environment reset helpers in `tests/setup.ts`
- [ ] T021 [P] Add PostgreSQL Testcontainers lifecycle and Prisma migration helpers in `tests/integration/database.ts`
- [ ] T022 [P] Add reusable authenticated request and test-user fixtures in `tests/fixtures/auth.ts`
- [ ] T023 Add authentication and session unit tests for hashing, token handling, expiration, revocation, inactive users, and generic failures in `tests/unit/auth.test.ts`
- [ ] T024 Add authorization unit tests for unauthenticated, operator, and administrator access decisions in `tests/unit/authorization.test.ts`
- [ ] T025 Add common validation unit tests for UUIDs, pagination, timestamps, date ranges, and malformed input in `tests/unit/validation.test.ts`
- [ ] T026 Configure strict TypeScript checks and confirm `tsconfig.json` rejects implicit unsafe types and `any`

**Checkpoint**: Database migrations, session guards, typed validation, response helpers, and test infrastructure are ready for story work.

---

## Phase 3: User Story 1 - Sign In to the Access System (Priority: P0) 🎯 MVP

**Goal**: Authorized users can authenticate, access protected pages/API routes, and sign out without exposing credentials or session secrets.

**Independent Test**: Use the login page and auth API with valid, invalid, expired, revoked, and inactive-user credentials; verify session creation, protected-route behavior, and logout.

### Tests for User Story 1

- [ ] T027 [P] [US1] Add Route Handler tests for login, logout, and current-user status responses in `tests/contract/auth-routes.test.ts`
- [ ] T028 [P] [US1] Add integration tests for persisted sessions, cookie flags, generic invalid-login errors, expiration, revocation, and inactive users in `tests/integration/auth-session.test.ts`
- [ ] T029 [P] [US1] Add login form component tests for labels, keyboard submission, loading state, validation feedback, and non-sensitive errors in `tests/components/login-form.test.tsx`
- [ ] T030 [P] [US1] Add Playwright coverage for valid login, invalid login, protected-route redirect, and logout in `tests/e2e/auth.spec.ts`

### Implementation for User Story 1

- [ ] T031 [US1] Implement `POST /api/auth/login` with boundary validation, generic failures, session creation, and secure cookie setting in `app/api/auth/login/route.ts`
- [ ] T032 [US1] Implement `POST /api/auth/logout` with current-session revocation and cookie clearing in `app/api/auth/logout/route.ts`
- [ ] T033 [US1] Implement `GET /api/auth/me` with server-side session resolution and safe user summary output in `app/api/auth/me/route.ts`
- [ ] T034 [US1] Create the accessible login page and server/client boundary in `app/(auth)/login/page.tsx`
- [ ] T035 [US1] Create the interactive login form with accessible labels, focus handling, pending state, and safe error display in `components/auth/login-form.tsx`
- [ ] T036 [US1] Add protected dashboard layout and server-side redirect/authorization handling in `app/(dashboard)/layout.tsx`
- [ ] T037 [US1] Add initial authenticated dashboard page with user summary and sign-out action in `app/(dashboard)/page.tsx`
- [ ] T038 [US1] Add authentication API error handling and session-safe cache headers in `lib/auth/auth-errors.ts`

**Checkpoint**: A valid user can sign in and reach the protected shell; invalid, signed-out, expired, revoked, and inactive sessions cannot access protected functionality.

---

## Phase 4: User Story 2 - Manage Employees and RFID Cards (Priority: P1)

**Goal**: Administrators can manage employee records, register/reassign/deactivate cards, and preserve card assignment history without exposing raw identifiers.

**Independent Test**: As an administrator, create/read/update/deactivate employees and cards, test duplicate/conflicting assignments, and verify the API and management views.

### Tests for User Story 2

- [ ] T039 [P] [US2] Add employee domain-service tests for required fields, normalization, duplicate identifiers, updates, deactivation, and historical references in `tests/unit/employee-service.test.ts`
- [ ] T040 [P] [US2] Add RFID card and assignment tests for digesting, unique cards, active assignment constraints, reassignment, deactivation, and raw-identifier redaction in `tests/unit/rfid-card-service.test.ts`
- [ ] T041 [P] [US2] Add Route Handler contract tests for employee and card CRUD, authorization, validation, conflicts, and safe response shapes in `tests/contract/employee-card-routes.test.ts`
- [ ] T042 [P] [US2] Add PostgreSQL integration tests for employee/card relations, assignment history, uniqueness, and restrictive delete behavior in `tests/integration/employee-card-persistence.test.ts`
- [ ] T043 [P] [US2] Add component tests for employee and card forms, accessible tables, loading/error states, and deactivation confirmation in `tests/components/employee-card-management.test.tsx`
- [ ] T044 [P] [US2] Add Playwright coverage for administrator employee/card CRUD and duplicate assignment handling in `tests/e2e/employee-card-management.spec.ts`

### Implementation for User Story 2

- [ ] T045 [P] [US2] Implement RFID digesting and safe card projection helpers in `lib/domain/rfid-card.ts`
- [ ] T046 [US2] Implement employee CRUD and deactivation services in `lib/domain/employees.ts`
- [ ] T047 [US2] Implement card registration, reassignment, deactivation, and assignment-history services in `lib/domain/rfid-cards.ts`
- [ ] T048 [US2] Implement `GET/POST /api/employees` with role authorization, validation, pagination, and conflict mapping in `app/api/employees/route.ts`
- [ ] T049 [US2] Implement `GET/PATCH /api/employees/[id]` with deactivation and historical-reference protection in `app/api/employees/[id]/route.ts`
- [ ] T050 [US2] Implement `GET/POST /api/rfid-cards` with raw-identifier hashing and safe projections in `app/api/rfid-cards/route.ts`
- [ ] T051 [US2] Implement `GET/PATCH /api/rfid-cards/[id]` with transactional assignment changes and deactivation in `app/api/rfid-cards/[id]/route.ts`
- [ ] T052 [P] [US2] Build employee management page and reusable components in `app/(dashboard)/employees/page.tsx` and `components/employees/`
- [ ] T053 [P] [US2] Build RFID card management page and reusable components in `app/(dashboard)/rfid-cards/page.tsx` and `components/rfid-cards/`
- [ ] T054 [US2] Add employee/card navigation and role-aware controls to `app/(dashboard)/layout.tsx`

**Checkpoint**: Administrators can manage employees and cards, while operators can only perform permitted reads and no raw RFID identifier leaves the server boundary.

---

## Phase 5: User Story 3 - Manage Access Doors (Priority: P1)

**Goal**: Administrators can manage active facility doors, and inactive doors remain available for historical references but cannot authorize new access.

**Independent Test**: As an administrator, create/read/update/deactivate doors, test duplicate codes and inactive-door behavior, and verify API/UI results.

### Tests for User Story 3

- [ ] T055 [P] [US3] Add door domain-service tests for required fields, stable unique codes, updates, deactivation, and duplicate conflicts in `tests/unit/door-service.test.ts`
- [ ] T056 [P] [US3] Add Route Handler contract tests for door CRUD, role authorization, validation, not-found, and conflict responses in `tests/contract/door-routes.test.ts`
- [ ] T057 [P] [US3] Add PostgreSQL integration tests for door uniqueness, event references, and restrictive delete behavior in `tests/integration/door-persistence.test.ts`
- [ ] T058 [P] [US3] Add component tests for accessible door forms, tables, status display, and deactivation errors in `tests/components/door-management.test.tsx`
- [ ] T059 [P] [US3] Add Playwright coverage for door CRUD and inactive-door behavior in `tests/e2e/door-management.spec.ts`

### Implementation for User Story 3

- [ ] T060 [US3] Implement door CRUD and deactivation services in `lib/domain/doors.ts`
- [ ] T061 [US3] Implement `GET/POST /api/doors` with role authorization, validation, pagination, and conflict mapping in `app/api/doors/route.ts`
- [ ] T062 [US3] Implement `GET/PATCH /api/doors/[id]` with historical-reference protection in `app/api/doors/[id]/route.ts`
- [ ] T063 [US3] Build the door management page and reusable accessible components in `app/(dashboard)/doors/page.tsx` and `components/doors/`
- [ ] T064 [US3] Add door navigation and administrator controls to `app/(dashboard)/layout.tsx`

**Checkpoint**: Active doors can be administered and selected for operations; inactive doors cannot be used for successful new access decisions.

---

## Phase 6: User Story 4 - Record an RFID Access Event (Priority: P0)

**Goal**: Authorized software clients can submit access attempts and receive trusted granted/denied decisions persisted with complete audit context.

**Independent Test**: Submit valid, inactive, unknown, mismatched, malformed, duplicate-retry, and persistence-failure requests to the REST endpoint and inspect stored events.

### Tests for User Story 4

- [ ] T065 [P] [US4] Add access-policy unit tests for active state, employee/card assignment, door state, unknown cards, mismatches, future timestamps, and failure reasons in `tests/unit/access-policy.test.ts`
- [ ] T066 [P] [US4] Add Route Handler contract tests for event creation status codes, request validation, authorization, idempotency, and safe output in `tests/contract/access-event-create.test.ts`
- [ ] T067 [P] [US4] Add PostgreSQL integration tests for transactional decision plus append-only persistence, complete relations, UTC timestamps, and duplicate idempotency keys in `tests/integration/access-event-persistence.test.ts`
- [ ] T068 [P] [US4] Add Playwright coverage for recording granted and denied software-submitted events through the UI flow in `tests/e2e/access-event-recording.spec.ts`

### Implementation for User Story 4

- [ ] T069 [US4] Implement access decision policy and typed failure categories in `lib/domain/access-policy.ts`
- [ ] T070 [US4] Implement transactional access-event recording with idempotency and UTC timestamp validation in `lib/domain/access-events.ts`
- [ ] T071 [US4] Implement `POST /api/access-events` with server-derived decisions, authorization, validation, and safe errors in `app/api/access-events/route.ts`
- [ ] T072 [US4] Build the access-event submission form with accessible employee/card/door selectors and decision feedback in `components/access-events/access-event-form.tsx`
- [ ] T073 [US4] Add software event recording controls and status/error states to `app/(dashboard)/access-events/page.tsx`
- [ ] T074 [US4] Add event recording navigation and role-aware controls to `app/(dashboard)/layout.tsx`

**Checkpoint**: Valid submissions persist complete granted/denied events only after trusted database evaluation; malformed requests create no events and raw RFID values remain protected.

---

## Phase 7: User Story 5 - Review and Filter Centralized Access Records (Priority: P2)

**Goal**: Authorized users can review paginated access events and combine employee, card, door, and UTC date-range filters with AND semantics.

**Independent Test**: Seed events across several entities and timestamps, apply each filter alone and in combination, and verify exact results plus explicit empty/error states.

### Tests for User Story 5

- [ ] T075 [P] [US5] Add filter-parser unit tests for UUID filters, half-open UTC date ranges, pagination bounds, invalid values, and empty filters in `tests/unit/access-event-filters.test.ts`
- [ ] T076 [P] [US5] Add Route Handler contract tests for combined filters, pagination, unauthorized access, validation errors, and response projections in `tests/contract/access-event-list.test.ts`
- [ ] T077 [P] [US5] Add PostgreSQL integration tests for indexed employee/card/door/date filtering, AND semantics, ordering, and zero-result queries in `tests/integration/access-event-query.test.ts`
- [ ] T078 [P] [US5] Add component tests for filter controls, accessible results table, loading/error/empty states, and keyboard operation in `tests/components/access-event-records.test.tsx`
- [ ] T079 [P] [US5] Add Playwright coverage for combined filtering, empty results, responsive layout, and signed-out isolation in `tests/e2e/access-event-records.spec.ts`

### Implementation for User Story 5

- [ ] T080 [US5] Implement typed access-event filter parsing and pagination in `lib/validation/access-event-filters.ts`
- [ ] T081 [US5] Implement paginated access-event query service with AND filters and safe projections in `lib/domain/access-event-queries.ts`
- [ ] T082 [US5] Implement `GET /api/access-events` with authorization, validated filters, pagination, and stable response shape in `app/api/access-events/route.ts`
- [ ] T083 [US5] Build filter controls with employee/card/door/date inputs and accessible reset behavior in `components/access-events/access-event-filters.tsx`
- [ ] T084 [US5] Build the centralized records table with date/time, decision, reason, empty, loading, and error states in `components/access-events/access-event-table.tsx`
- [ ] T085 [US5] Complete the records page data loading, URL filter state, pagination, and responsive layout in `app/(dashboard)/access-events/page.tsx`

**Checkpoint**: Authorized users can investigate centralized event history with exact combined filters, while unauthorized users and invalid queries receive no protected data.

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Complete shared accessibility, security, documentation, performance, and release-quality verification across all stories.

- [ ] T086 [P] Add global responsive/accessibility styles and focus-visible states using Tailwind utilities in `app/globals.css`
- [ ] T087 [P] Add page metadata, descriptive headings, route announcements, and accessible navigation labels in `app/layout.tsx` and dashboard pages
- [ ] T088 [P] Add centralized authorization/error/security logging review checklist to `docs/security-review.md`
- [ ] T089 [P] Document environment variables, Prisma migration workflow, seeded administrator setup, and smoke tests in `README.md`
- [ ] T090 [P] Add Prisma seed data for a development administrator, employees, cards, doors, and representative access events in `prisma/seed.ts`
- [ ] T091 Add database indexes/query profiling for access-record filters and verify the SC-004 three-second target in `tests/integration/access-event-query.performance.test.ts`
- [ ] T092 Add end-to-end keyboard, responsive mobile viewport, and accessible form-label verification in `tests/e2e/accessibility.spec.ts`
- [ ] T093 Run the complete lint, strict typecheck, unit/integration/contract/component tests, Playwright suite, and production build from `quickstart.md`
- [ ] T094 Review all API response projections and logs for raw RFID identifiers, password hashes, session tokens, and stack traces before release
- [ ] T095 Verify OpenAPI contract synchronization with implemented Route Handlers in `specs/001-rfid-access-control/contracts/openapi.yaml`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1: Setup** has no dependencies and can begin immediately.
- **Phase 2: Foundational** depends on Phase 1 and blocks all user stories.
- **Phase 3: US1** depends on Phase 2 and establishes authenticated access for later stories.
- **Phase 4: US2** depends on Phase 2 and requires US1's authorization guards for protected administration.
- **Phase 5: US3** depends on Phase 2 and requires US1's authorization guards for protected administration.
- **Phase 6: US4** depends on Phase 2 plus the employee/card foundation from US2 and door foundation from US3.
- **Phase 7: US5** depends on Phase 6 event persistence and the entity data from US2 and US3.
- **Phase 8: Polish** depends on all desired user stories being complete.

### User Story Dependencies

- **US1 (P0)**: Starts after Phase 2; independent MVP for authentication and protected shell.
- **US2 (P1)**: Starts after Phase 2; uses US1 auth guards but does not depend on event recording.
- **US3 (P1)**: Starts after Phase 2; uses US1 auth guards but does not depend on US2.
- **US4 (P0)**: Depends on US2's employee/card and assignment services plus US3's door services.
- **US5 (P2)**: Depends on US4's persisted event shape and queryable entity relationships.

### Within Each User Story

- Write focused tests before implementation and confirm they fail for missing behavior where practical.
- Implement domain models/services before Route Handlers and UI integration.
- Keep all Route Handler validation and authorization at the boundary.
- Complete and independently validate each story at its checkpoint before widening scope.

### Parallel Opportunities

- Phase 1 tasks T002-T007 can run in parallel after T001 where files do not overlap.
- Foundational helpers T011-T014, T017-T022 can run in parallel; T015-T016 depend on the auth primitives.
- US2 tests T039-T044 can run in parallel; domain services T045-T047 can be split by file.
- US3 tests T055-T059 can run in parallel; service and UI work can proceed after the schema is available.
- US4 tests T065-T068 can run in parallel after the shared fixtures exist.
- US5 tests T075-T079 can run in parallel after seeded event fixtures exist.
- Cross-cutting documentation, styling, seed, and accessibility tasks T086-T090 can run in parallel with performance and contract review once their referenced surfaces exist.

## Parallel Example: User Story 1

```text
Task T027: Route Handler tests in tests/contract/auth-routes.test.ts
Task T028: Session integration tests in tests/integration/auth-session.test.ts
Task T029: Login form tests in tests/components/login-form.test.tsx
Task T030: Auth E2E tests in tests/e2e/auth.spec.ts
```

## Parallel Example: User Story 2

```text
Task T039: Employee service tests in tests/unit/employee-service.test.ts
Task T040: RFID card service tests in tests/unit/rfid-card-service.test.ts
Task T041: Employee/card contract tests in tests/contract/employee-card-routes.test.ts
Task T042: Employee/card persistence tests in tests/integration/employee-card-persistence.test.ts
Task T043: Employee/card component tests in tests/components/employee-card-management.test.tsx
Task T044: Employee/card E2E tests in tests/e2e/employee-card-management.spec.ts
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 setup and Phase 2 foundation.
2. Complete US1 authentication and protected shell.
3. Complete US2 employee/card administration and US3 door administration.
4. Complete US4 event recording.
5. Complete US5 centralized filtered review.
6. Stop at each checkpoint and run the story's focused tests before continuing.

### Incremental Delivery

1. Foundation ready: migrations, sessions, guards, validation, and test harness.
2. US1: secured login and protected shell demonstrable independently.
3. US2 + US3: accurate employees/cards/doors demonstrable independently after auth.
4. US4: software-submitted granted/denied event recording demonstrable independently.
5. US5: searchable centralized audit history demonstrable independently.
6. Polish: accessibility, security review, performance, docs, and full release gates.

### Parallel Team Strategy

1. One developer completes Phase 1 and Phase 2 together because the database and auth boundaries are shared.
2. After Phase 2, one developer owns US1 while separate developers can work on US2 and US3.
3. US4 begins after the employee/card/door service contracts stabilize.
4. US5 and cross-cutting quality work follow the event contract and seeded fixtures.

## Traceability Summary

| User story | Primary requirements | Independent validation |
|---|---|---|
| US1 Authentication | FR-001, FR-002, FR-018, FR-021 | T027-T038, `tests/e2e/auth.spec.ts` |
| US2 Employees and RFID cards | FR-003 through FR-007, FR-019 | T039-T054, `tests/e2e/employee-card-management.spec.ts` |
| US3 Doors | FR-008, FR-009, FR-019 | T055-T064, `tests/e2e/door-management.spec.ts` |
| US4 Access event recording | FR-010 through FR-013, FR-017, FR-018 | T065-T074, `tests/e2e/access-event-recording.spec.ts` |
| US5 Centralized filtering | FR-014 through FR-016, FR-020 | T075-T085, `tests/e2e/access-event-records.spec.ts` |
