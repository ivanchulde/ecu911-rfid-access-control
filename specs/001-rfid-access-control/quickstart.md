# Quickstart: ECU 911 RFID Access Control System

## Prerequisites

- Node.js compatible with Next.js 16.3.5
- npm
- PostgreSQL 15+ available locally or through Docker/Testcontainers
- Git

## Install dependencies

From the repository root:

```bash
npm install
npm install @prisma/client zod argon2
npm install -D prisma vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event @playwright/test testcontainers
npx playwright install
```

Keep dependency additions in the feature branch and review the resulting lockfile changes.

## Configure the database

Create a local PostgreSQL database and add a `.env.local` file that is ignored by Git:

```dotenv
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/ecu911_rfid_access"
SESSION_COOKIE_NAME="session"
SESSION_TTL_DAYS="7"
RFID_HASH_SECRET="replace-with-a-development-only-secret"
```

Never commit real credentials, production database URLs, session secrets, or raw RFID identifiers.

## Initialize Prisma

After `prisma/schema.prisma` and the first migration are implemented:

```bash
npx prisma generate
npx prisma migrate dev --name init
```

Use `npx prisma migrate deploy` in deployment environments. Do not use `prisma db push` for production schema changes.

## Run the application

```bash
npm run dev
```

Open `http://localhost:3000`. The login page is the entry point; protected routes and API handlers require a valid server-managed session.

## Verification commands

```bash
npm run lint
npx tsc --noEmit
npm test
npm run test:e2e
npm run build
```

A complete MVP verification should cover:

1. Valid, invalid, expired, revoked, and inactive-user authentication.
2. Admin-only employee, RFID card, and door CRUD/deactivation.
3. Duplicate identifiers and invalid request validation.
4. Granted and denied software-submitted access events.
5. Employee/card/door/date-range filters combined with `AND` semantics.
6. No raw RFID identifiers in responses, URLs, logs, or error messages.
7. Keyboard navigation, labels, focus states, responsive layouts, and explicit empty/error states.

## API smoke examples

After signing in through the UI or a test client that preserves the session cookie:

```bash
curl -i -b cookies.txt -c cookies.txt -X POST http://localhost:3000/api/access-events \
  -H 'Content-Type: application/json' \
  -d '{"employeeId":"<employee-uuid>","rfidCardId":"<card-uuid>","doorId":"<door-uuid>","occurredAt":"2026-09-19T12:00:00Z"}'

curl -i -b cookies.txt 'http://localhost:3000/api/access-events?doorId=<door-uuid>&start=2026-09-19T00:00:00Z&end=2026-09-20T00:00:00Z'
```

The event endpoint derives the decision from trusted database state. Clients do not submit an authoritative `GRANTED` value.
