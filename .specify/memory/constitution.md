<!--
Sync Impact Report
- Version change: template/unratified -> 1.0.0
- Modified principles: all five core principles are newly ratified
- Added sections: Technical and Security Constraints; Development Workflow and Quality Gates
- Removed sections: none
- Templates requiring updates: plan-template.md (✅ generic Constitution Check remains applicable)
	spec-template.md (✅ required user stories, requirements, and success criteria align)
	tasks-template.md (✅ supports security, API, database, testing, and accessibility tasks)
	.specify/templates/commands/*.md (✅ no commands directory exists in this repository)
- Follow-up TODOs: none.
-->

# ECU 911 RFID Access Control System Constitution

## Core Principles

### I. Type-Safe Implementation

All application code MUST use TypeScript in strict mode. The codebase MUST NOT use
`any`; unknown external data MUST be narrowed through explicit validation or typed
adapters. Developers MUST define clear types and interfaces at module boundaries and
use meaningful, domain-oriented names. Variables, functions, components, routes, and
database fields MUST follow consistent naming: `camelCase` for values and functions,
`PascalCase` for types and components, and `kebab-case` for URL path segments. This
keeps access-control behavior explicit and reduces defects caused by implicit data shapes.

### II. Next.js and Presentation Boundaries

The application MUST use Next.js App Router and file-based routing. Server Components
are the default; Client Components MUST be introduced only for browser APIs, local
interactive state, or event handlers that cannot run on the server. Business rules,
authorization, persistence, and integration logic MUST remain separate from presentation
when practical. Tailwind CSS MUST be used with a utility-first approach; custom CSS is
allowed only when Tailwind cannot express the required behavior or when a shared design
primitive makes it necessary. This preserves predictable rendering and maintainable UI.

### III. Secure and Auditable Access

Protected functionality MUST require authenticated users and explicit authorization based
on role and permission. The system MUST deny by default and MUST fail closed when identity,
permission, card, door, or policy data is missing or invalid. Every RFID access event MUST
record the employee, RFID card, door, date/time, decision, and a reason or failure category.
Event records MUST be append-oriented and raw card identifiers MUST NOT be exposed in
client-visible errors, URLs, or ordinary logs. These rules make facility access decisions
both defensible and reviewable.

### IV. Validated REST Contracts

All REST APIs MUST be implemented with Next.js Route Handlers and MUST validate request
headers, path parameters, query parameters, and bodies at the boundary. Endpoints MUST
return appropriate HTTP status codes, stable response shapes, and clear error responses
without leaking secrets or internal stack traces. Authentication and authorization MUST
be enforced server-side for every protected route, not only in the UI. API changes MUST
document affected contracts and preserve backward compatibility unless a reviewed change
explicitly permits a breaking change.

### V. Verification Before Delivery

Every feature MUST define independently testable user scenarios and measurable acceptance
criteria. Changes MUST include focused tests for authentication, authorization, CRUD
operations, REST endpoints, input validation, expected failures, and relevant RFID event
behavior. A change is not ready for review until linting, type checking, the relevant test
suite, and a production build pass or the exception is documented. This makes the system's
security and operational behavior verifiable rather than assumed.

## Technical and Security Constraints

The required stack is Next.js with App Router, TypeScript strict mode, Tailwind CSS,
PostgreSQL, Prisma ORM, and REST APIs implemented with Next.js Route Handlers. Prisma
models MUST represent clear relationships among employees, RFID cards, doors, and access
events, including appropriate uniqueness, required fields, and referential behavior.
Schema changes MUST be reviewed for migration safety and data integrity.

Authentication credentials, session data, database URLs, and other secrets MUST come
from environment or secret-management facilities and MUST NOT be committed to source
control. RFID identifiers and personal data MUST be minimized in logs and responses.
Authorization checks MUST be performed against trusted server-side data.

User-facing pages MUST be responsive across supported mobile and desktop sizes. Interactive
controls MUST have accessible names, keyboard access, visible focus states, sufficient
contrast, and semantic HTML. Forms and errors MUST be understandable to assistive
technology. Accessibility is part of feature acceptance, not optional polish.

## Development Workflow and Quality Gates

Work MUST be developed on a dedicated feature branch; direct pushes to `main` are
prohibited. Every change MUST use a pull request with a clear description, linked feature
or issue context, verification results, and reviewers who check security, data integrity,
API behavior, accessibility, and maintainability as applicable. At least one code review
approval is required before merge, and CI checks MUST pass.

Specifications MUST define prioritized user stories, edge cases, functional requirements,
and measurable success criteria. Plans MUST include a Constitution Check before design and
again after design. Tasks MUST identify database migrations, authentication, validation,
API contracts, tests, and accessibility work whenever those concerns are affected.

## Governance
<!-- Example: Constitution supersedes all other practices; Amendments require documentation, approval, migration plan -->

This constitution supersedes conflicting project practices. Amendments MUST state the
reason for the change, update the Sync Impact Report, propagate affected guidance to
dependent templates, and update the version and last-amended date. Versioning follows
Semantic Versioning: MAJOR for incompatible principle changes or removals, MINOR for new
principles or materially expanded obligations, and PATCH for clarifications that do not
change obligations.

Every pull request and release review MUST verify compliance with this constitution.
Exceptions MUST be documented with the affected rule, justification, owner, mitigation,
and expiration or review date. Unresolved authentication, authorization, privacy, or data
integrity exceptions block release unless explicitly accepted by the project owner.

**Version**: 1.0.0 | **Ratified**: 2026-09-19 | **Last Amended**: 2026-09-19
<!-- Example: Version: 2.1.1 | Ratified: 2025-06-13 | Last Amended: 2025-07-16 -->
