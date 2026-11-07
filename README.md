# CargoLens AI

AI-powered import documentation intelligence for a customs brokerage and freight-forwarding desk at Nhava Sheva. Documentation executives upload a Commercial Invoice, Packing List and Bill of Lading; CargoLens extracts structured shipment data with per-field confidence and evidence, cross-checks the three documents against each other with a deterministic rules engine, and routes the shipment through human review and manager approval before it can be exported.

**Core principle:** AI extracts. Deterministic rules validate. Humans approve. The AI never silently writes to an approved record, and no document can be exported while a critical, unresolved discrepancy remains open.

See [`TECHNICAL_DESIGN.md`](TECHNICAL_DESIGN.md) for architecture, data model and the AI/validation pipeline, and [`DEPLOYMENT.md`](DEPLOYMENT.md) to run it locally.

## Stack

- **Frontend:** Next.js 16 (App Router), React 19, TypeScript (strict), Tailwind CSS v4, Radix UI primitives, TanStack Query, React Hook Form + Zod, pdf.js.
- **Backend:** Node.js, Express, TypeScript, Prisma ORM, PostgreSQL.
- **AI:** Gemini multimodal API behind a provider abstraction, with an explicitly labeled deterministic demo adapter as a fallback when no API key is configured.
- **Testing:** Vitest (unit + API integration tests via Supertest), Playwright (end-to-end).

## Monorepo layout

```
apps/
  web/     Next.js frontend
  api/     Express API, Prisma schema/migrations, AI providers, validation engine
packages/
  shared/  Zod schemas, enums and types shared by both apps (compiled to dist/)
```

## Quickstart

```bash
pnpm install
pnpm --filter @cargolens/shared build   # shared types are consumed as compiled dist/, not raw .ts
cp apps/api/.env.example apps/api/.env   # fill in DATABASE_URL and JWT_SECRET
cp apps/web/.env.example apps/web/.env.local
pnpm --filter api prisma:migrate
pnpm --filter api db:seed
pnpm --filter api dev     # http://localhost:4000
pnpm --filter web dev     # http://localhost:3000
```

Demo logins (created by the seed script):

| Role | Email | Password |
|---|---|---|
| Documentation Executive | `executive@cargolens.demo` | `Executive123!` |
| Documentation Manager | `manager@cargolens.demo` | `Manager123!` |

Full instructions, including database setup options and environment variables, are in [`DEPLOYMENT.md`](DEPLOYMENT.md).

## Tests

```bash
pnpm --filter api test          # Vitest: validation-engine unit tests + API integration tests
pnpm --filter web exec playwright test   # Playwright end-to-end lifecycle test
```

The API integration suite runs against a separate `cargolens_test` database (see `apps/api/.env.test`) so it never touches demo data.
