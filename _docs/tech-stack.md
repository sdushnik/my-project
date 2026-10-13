# Tech stack — Option A (locked)

**Decision date:** 13 Oct 2026  
**Product baseline:** [_docs/plan.md](./plan.md)  
**Development spec:** [_docs/spec.md](./spec.md)

## Chosen stack

| Layer | Choice |
|---|---|
| App | Next.js App Router, TypeScript |
| Database | PostgreSQL |
| ORM | Drizzle |
| Auth | Auth.js — email/password + Google |
| UI | Tailwind CSS + shadcn/ui |
| Validation | Zod |
| Timezones | Luxon; household stores an IANA timezone name |
| Hosting | Vercel (app) + Neon (Postgres) |

## Why these defaults

- **PostgreSQL** is required for unique check-ins, 7-day corrections, weekly windows, and challenge percentages.
- **Drizzle** keeps scoring and schedule queries as explicit SQL instead of hiding them behind a heavy ORM.
- **Auth.js** covers email/password and Google on the Next.js server.
- **shadcn/ui** covers a responsive, accessible dashboard without a custom design system.
- Privacy and authorization live in **server queries and mutations**, not in hidden UI.

## Out of MVP

Redis, job queues, GraphQL, email delivery, browser push, Realtime, and native apps.

Challenge results and badge eligibility may be computed on read until volume requires a worker.
