# Yabantu

> One curriculum. Eleven languages. Every child deserves to understand.

This repository is the Sprint 1 foundation described in the Yabantu Master Document (October 2026): repository, database migrations, authentication, roles, learner profile, parent–learner links, language foundation, onboarding and tests. Sprint 2 adds lessons in several languages, server-scored assessments, learner progress, a parent progress view and content approval.

**Status: written, not yet run.** The code in this repository was written without access to the npm registry, so it has **not** been installed, compiled, linted or tested. Treat the first `npm install && npm test` as the first real check, and expect to fix some small errors. See [docs/product/sprint-1-status.md](docs/product/sprint-1-status.md) for the ticket-by-ticket position.

## What is here

| Path | What it is |
| --- | --- |
| `apps/api` | NestJS REST API (`/api/v1`): auth, roles, learner, parent, admin, reference data |
| `apps/mobile` | Expo / React Native app: welcome, account type, register, login, forgot password, learner onboarding, role home screens |
| `packages/types` | Shared constants and API types (roles, the 11 languages, error codes, DTOs) |
| `packages/validation` | Shared zod schemas, used by the API to enforce and by the app for instant feedback |
| `database/migrations` | Forward-only SQL migrations |
| `database/seeds` | Idempotent reference data (11 languages, grades, subjects) and 7 demo lessons generated from the prototype by `scripts/gen-content-seed.js` |
| `docs/` | API reference, architecture, security notes, decision log, Sprint 1 status |
| `prototype/yabantu.html` | The clickable product prototype (demo data only) |

## Quick start

Requirements: Node 20.6 or newer, Docker (for PostgreSQL).

```bash
docker compose up -d                 # PostgreSQL on :5432
cp .env.example .env                 # then set JWT_SECRET (see the comment in the file)
npm install
npm run db:migrate                   # create tables
npm run db:seed                      # languages, grades, subjects, demo lessons (as drafts)
npm run db:publish-demo              # development only: make the demo lessons visible to learners
npm run dev:api                      # API on http://localhost:3000/api/v1  (docs at /api/docs)
```

Create the first administrator (there is no self-registration for staff roles):

```bash
ADMIN_EMAIL=you@example.com ADMIN_PASSWORD='a-long-passphrase' npm run db:create-admin
```

Run the mobile app (set the API address for your device or emulator):

```bash
EXPO_PUBLIC_API_URL=http://<your-computer-ip>:3000/api/v1 npm run dev:mobile
```

Expo SDK versions move quickly. If `expo start` complains about versions, run `npx expo install --fix` inside `apps/mobile`.

## Tests, lint, types

```bash
export TEST_DATABASE_URL=postgres://yabantu:yabantu@localhost:5432/yabantu
npm test            # builds shared packages, then runs the API tests against real PostgreSQL
npm run lint
npm run typecheck
```

Each test file creates its own throwaway schema and drops it afterwards, so tests never touch your development data. Without `TEST_DATABASE_URL`, API tests stop with a clear message.

## Rules of the road

- Secrets are never committed. `.env` is git-ignored; `.env.example` holds placeholders only.
- Do not enter real children's data into any environment until the privacy and safeguarding review in [docs/security](docs/security/child-safety-and-privacy.md) is complete.
- Database changes are new migration files, never edits to an applied one.
- Every protected endpoint checks authentication, role and, where a record belongs to someone, ownership.
