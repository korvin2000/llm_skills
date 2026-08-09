# AGENTS.md

Payments service. Go 1.23, Postgres, deployed via Argo.

## Commands

| Task | Command |
|---|---|
| Test | `go test ./...` |
| Test one package | `go test ./internal/ledger -run TestName` |
| Lint | `golangci-lint run` (config is the source of truth for style) |
| Generate mocks | `go generate ./...` after changing any interface |
| Local DB | `docker compose up -d postgres` |

## Invariants

- Money is `int64` minor units. Never `float64` — rounding drift is unrecoverable once written.
- Ledger rows are immutable. Corrections are new compensating entries, so history stays replayable.
- Every handler takes `ctx` first and honours cancellation.

## Gotchas

- `internal/ledger/balance.go` caches per-account totals in Redis with no TTL. Invalidate on write
  or reads go stale silently.
- The Stripe sandbox rejects amounts ending in `13`. Test fixtures avoid them on purpose.

## Boundaries

NEVER run migrations against prod; Argo does it on merge.
NEVER commit to `main`; it is protected.
ASK before adding a dependency — the vendor directory is reviewed.

## Read when needed

| Topic | File | When |
|---|---|---|
| Ledger invariants | `docs/ledger-model.md` | before changing any balance calculation |
| Webhook retry policy | `docs/webhooks.md` | before touching `internal/webhook/` |

## Done

- `go test ./...` and `golangci-lint run` both pass.
- New endpoints have a contract test in `internal/api/contract_test.go`.
