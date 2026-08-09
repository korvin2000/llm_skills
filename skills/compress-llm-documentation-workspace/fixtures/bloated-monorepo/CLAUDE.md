# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Introduction and Welcome

Welcome to the Acme Platform monorepo! This document is intended to help AI assistants understand
the structure and conventions of our codebase so that they can be maximally helpful when making
changes. Please read it carefully before you start working, and always follow best practices when
writing code. We value clean, readable, maintainable, production-ready code above all else.

This is a TypeScript monorepo managed with pnpm workspaces. TypeScript is a strongly typed
programming language that builds on JavaScript by adding static type definitions. A monorepo is a
software development strategy in which the code for many projects is stored in the same
repository. pnpm is a fast, disk space efficient package manager that uses a content-addressable
store for node_modules.

## Repository Structure

The repository is organized as follows:

```
acme-platform/
├── apps/
│   ├── web/              # Next.js customer-facing app
│   ├── admin/            # internal admin dashboard
│   └── worker/           # background job processor
├── packages/
│   ├── ui/               # shared React component library
│   ├── db/               # Prisma schema and client
│   ├── config/           # shared eslint/tsconfig/tailwind presets
│   └── sdk/              # generated API client
├── services/
│   ├── api/              # Fastify REST API
│   └── events/           # Kafka consumers
├── infra/                # Terraform
├── scripts/
└── docs/
```

Each app in `apps/` is independently deployable. Each package in `packages/` is consumed by one or
more apps. The `services/` directory contains long-running backend services.

## Installed Dependencies

The monorepo currently uses the following major dependencies:

- next 15.0
- react 19.0
- typescript 5.6
- prisma 5.20
- fastify 5.0
- vitest 2.1
- playwright 1.48
- eslint 9.13
- prettier 3.3
- turbo 2.2

## Getting Started

To get started, first clone the repository. Then install dependencies by running `pnpm install`.
You will need Node 22 or later. Note that Node 24 is not currently supported because a transitive
dependency of the Prisma client segfaults on it — this cost us most of a day, so please do not
"helpfully" upgrade Node.

After installing, generate the Prisma client with `pnpm db:generate`. Then you can start the dev
servers with `pnpm dev`, which runs all apps in parallel via Turborepo.

## Coding Standards and Conventions

We care deeply about consistency. Please observe the following conventions at all times:

- Indent with 2 spaces. Never use tabs.
- Maximum line length is 100 characters.
- Use single quotes for strings in TypeScript, double quotes in JSON.
- Always add semicolons at the end of statements.
- Use `PascalCase` for React components, `camelCase` for functions and variables, and
  `SCREAMING_SNAKE_CASE` for constants.
- Prefer named exports over default exports.
- Sort imports: node builtins, then external packages, then internal `@acme/*` packages, then
  relative imports.
- Do not leave unused variables or imports.
- Add trailing commas in multiline literals.
- Prefer `const` over `let`, and never use `var`.

All of this is enforced by `packages/config/eslint.config.js` and Prettier. You can check your work
by running `pnpm lint`, and auto-fix most issues with `pnpm lint:fix`.

## TypeScript Guidelines

Write good types. Avoid `any` wherever possible; prefer `unknown` and narrow. Use discriminated
unions for state machines. Export types alongside the values they describe. Try to make illegal
states unrepresentable. Consider using branded types for IDs where it makes sense.

Run `pnpm typecheck` to check types across the whole workspace.

## Testing

Testing is extremely important and you should generally write tests for everything you change.

We use Vitest for unit and integration tests, and Playwright for end-to-end tests. Unit tests live
next to the code they test as `*.test.ts` files. End-to-end tests live in `apps/web/e2e/`.

Run the full test suite with `pnpm test`. Run a single package's tests with
`pnpm --filter @acme/db test`. Run the end-to-end tests with `pnpm test:e2e`, but note that this
requires the dev server to be running first.

It is usually a good idea to run tests before committing. CI will run them anyway.

## Database and Migrations

The Prisma schema lives at `packages/db/prisma/schema.prisma`. After editing it you must run
`pnpm db:generate` to regenerate the client, and `pnpm db:migrate` to create a migration.

Migrations are append-only. Production replays the full migration history from an empty database
during disaster recovery, so editing an already-applied migration will silently produce a different
schema than production has.

## The Generated SDK

`packages/sdk/src/generated/` is generated from the OpenAPI spec by `pnpm sdk:generate`. Never edit
files in that directory by hand — they are overwritten on every build, and people have lost work
this way more than once.

## API Error Format

Every API error response must use exactly this shape, because the mobile clients parse it
positionally and a change breaks shipped versions:

```json
{"error": {"code": "SNAKE_CASE_CODE", "message": "human readable", "requestId": "uuid"}}
```

## Git and Branching

Our branching model is fairly conventional. Feature work goes on a branch named `feat/<ticket-id>`,
and bug fixes go on `fix/<ticket-id>`, where the ticket ID is the Linear identifier such as
`ACME-1234`. When your work is ready you should open a pull request against `main`. Direct pushes
to `main` are blocked by branch protection, so you would not get very far anyway. Pull requests
require one approval and a green CI run before they can be merged. We use squash merges, so please
write a reasonable PR title because it becomes the commit message.

For urgent production fixes there is a separate hotfix process which involves branching from the
latest release tag rather than from `main`, but this is rare and a human will usually drive it.

## Deployment

Deployment is handled by Argo CD, which watches the `main` branch and syncs automatically. Do not
deploy manually. If something needs an emergency rollback, ask in `#platform-oncall` — do not run
`argocd app rollback` yourself unless you have been explicitly asked to.

## Environment Variables

The apps read configuration from the environment. The important ones are `DATABASE_URL`,
`REDIS_URL`, `KAFKA_BROKERS`, `STRIPE_SECRET_KEY`, and `SENTRY_DSN`. Copy `.env.example` to `.env`
to get started locally.

## Performance Notes

Try to keep bundle sizes reasonable. Be mindful of what you import into the client bundle. Avoid
unnecessary re-renders. Use `React.memo` where appropriate. Consider code splitting for large
routes, etc.

## Documentation

See docs/architecture.md

Also see docs/runbooks/

## A Note on Communication

Please be thorough and think step by step when working on tasks in this repository. If you are
unsure about something, it is better to ask than to guess. Try to explain your reasoning.

## Testing (repeated for emphasis)

Remember: run `pnpm test` before you commit. Testing is important.
