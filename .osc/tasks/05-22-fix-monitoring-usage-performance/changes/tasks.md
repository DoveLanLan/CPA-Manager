# Tasks: Fix Monitoring Usage Performance

- Date: 2026-05-22
- Owner(s): DoveLanLan
- Related: proposal.md, spec.md

## Assumptions

- The monitoring dashboard does not need `usage.Event.RawJSON`; export/import remains the only path that needs raw event snapshots.
- The existing UI consumes the `usage.BuildPayload` aggregate and detail fields, not raw event JSON.

## Checklist

- [x] 1) Optimize store query path
  - Target: `usage-service/internal/store/store.go`, `usage-service/internal/store/store_test.go`
  - Change: Add timestamp/id composite index and a dashboard recent-events method that omits `raw_json`.
  - Verify: `go test ./internal/store`.

- [x] 2) Use optimized path in usage handler
  - Target: `usage-service/internal/httpapi/server.go`
  - Change: Use the dashboard recent-events method for `GET /v0/management/usage`; keep export on full `RecentEvents`.
  - Verify: `go test ./internal/httpapi`.

- [x] 3) Prevent overlapping frontend usage refresh
  - Target: `src/services/api/usageService.ts`, `src/features/monitoring/hooks/useUsageData.ts`
  - Change: Add optional request `signal` support and make `loadUsage` single-flight/cancel stale requests.
  - Verify: `npm run type-check`, `npm run lint`, `npm run build`.

- [x] 4) Run full gates
  - Target: repository root and `usage-service`
  - Change: Run frontend gates, Usage Service tests, and diff hygiene.
  - Verify: `npm run type-check`, `npm run lint`, `VERSION=ci npm run build`, `go test ./...`, `git diff --check`.

## Notes

- Keep `USAGE_QUERY_LIMIT=100` on the current VPS until the new image is built and deployed.
- Local gates passed for focused store/http tests, full Usage Service tests, frontend type-check/lint/build/tests, Docker image build, and diff hygiene.
