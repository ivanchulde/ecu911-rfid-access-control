# Data Model: ECU 911 RFID Access Control System

## User

Represents an authenticated operator of the system.

| Field | Type | Rules |
|---|---|---|
| id | UUID | Primary key |
| email | string | Normalized lowercase; unique; required |
| passwordHash | string | Argon2id hash; never exposed |
| role | enum | `ADMIN` or `OPERATOR`; required |
| isActive | boolean | Required; inactive users cannot create sessions |
| createdAt | timestamp | UTC; required |
| updatedAt | timestamp | UTC; required |

Relationships: one user has many sessions. A user may be retained as the actor who submitted an event if operator attribution is needed, but access events remain about the employee/card/door decision.

## Session

Represents a revocable authenticated browser session.

| Field | Type | Rules |
|---|---|---|
| id | UUID | Primary key |
| userId | UUID | Required foreign key to User |
| tokenHash | string | Unique; hash of opaque random cookie token |
| expiresAt | timestamp | UTC; required |
| lastSeenAt | timestamp | UTC; required |
| revokedAt | timestamp | Nullable; revoked sessions are invalid |
| createdAt | timestamp | UTC; required |

Relationship: many sessions belong to one user. Session cookies contain only the opaque token, never the database identifier or token hash.

## Employee

Represents a person whose facility access is managed.

| Field | Type | Rules |
|---|---|---|
| id | UUID | Primary key |
| employeeNumber | string | Required; normalized; unique |
| fullName | string | Required; validated length |
| email | string | Optional; normalized if present |
| isActive | boolean | Required; deactivation preserves history |
| createdAt | timestamp | UTC; required |
| updatedAt | timestamp | UTC; required |

Relationships: one employee may have many card assignments and access events.

## RfidCard

Represents an RFID credential without exposing the raw identifier.

| Field | Type | Rules |
|---|---|---|
| id | UUID | Primary key |
| identifierDigest | string | Required; unique deterministic server-side digest |
| identifierLast4 | string | Optional display aid; never sufficient for authorization |
| isActive | boolean | Required; deactivation preserves history |
| createdAt | timestamp | UTC; required |
| updatedAt | timestamp | UTC; required |

Relationships: one card has many assignment-history records and access events. The current assignment is represented by one active `RfidCardAssignment` row.

## RfidCardAssignment

Preserves card ownership over time and prevents historical ambiguity after reassignment.

| Field | Type | Rules |
|---|---|---|
| id | UUID | Primary key |
| rfidCardId | UUID | Required foreign key to RfidCard |
| employeeId | UUID | Required foreign key to Employee |
| assignedAt | timestamp | UTC; required |
| unassignedAt | timestamp | Nullable; active assignment has null value |

Constraints: at most one active assignment per card, assignment intervals must not overlap, and assignment changes must be authorized and transactional.

## Door

Represents a controlled facility entry point.

| Field | Type | Rules |
|---|---|---|
| id | UUID | Primary key |
| code | string | Required; stable and unique |
| name | string | Required; validated length |
| location | string | Required or documented optional field |
| isActive | boolean | Required; deactivation preserves history |
| createdAt | timestamp | UTC; required |
| updatedAt | timestamp | UTC; required |

Relationships: one door has many access events.

## AccessEvent

Append-oriented record of an access attempt. No update or delete operation is exposed through the MVP API.

| Field | Type | Rules |
|---|---|---|
| id | UUID | Primary key |
| employeeId | UUID | Required foreign key to Employee when an employee is known |
| rfidCardId | UUID | Nullable for an unknown card; foreign key when known |
| doorId | UUID | Required foreign key to Door |
| decision | enum | `GRANTED` or `DENIED`; required |
| reason | enum/string | Required failure or decision category |
| occurredAt | timestamp | UTC; timezone-qualified input; required |
| recordedAt | timestamp | UTC persistence time; required |
| idempotencyKey | string | Optional unique key for retry-safe submissions |

Relationships: each event references an employee, optional known card, and door. Foreign keys use restrictive delete behavior. Unknown-card denied attempts may retain a non-reversible digest in a separate audit field if required, but raw identifiers are never exposed.

## State and Integrity Rules

- User, employee, card, and door lifecycle changes use `isActive = false`; historical event rows remain queryable.
- Successful authorization requires an active employee, active card, active door, and active assignment matching the employee.
- Unknown, inactive, or mismatched credentials result in a denied event when sufficient data exists to audit the attempt; malformed requests produce validation errors and no event.
- All event timestamps are stored in UTC PostgreSQL `timestamptz(3)`. Query date filters use a half-open range `[start, end)`.
- Recommended indexes: `AccessEvent(occurredAt)`, `(employeeId, occurredAt)`, `(rfidCardId, occurredAt)`, `(doorId, occurredAt)`, and unique identifiers for users, employees, cards, and doors.
- Event relations use `Restrict`/`NoAction` delete behavior. No cascade may remove historical access records.
