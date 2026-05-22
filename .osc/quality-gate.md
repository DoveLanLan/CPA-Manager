# Quality Gate

- Date: 2026-05-22
- Scope: Fix monitoring usage performance in CPA-Manager.

## Assumptions:

- The monitoring dashboard does not need `usage.Event.RawJSON`.
- Export/import remains the compatibility path that needs full raw event snapshots.
- The VPS keeps the existing `USAGE_QUERY_LIMIT=100` mitigation until the optimized image is deployed.

## Suspected Change Scope:

- `usage-service/internal/store/store.go`
- `usage-service/internal/store/store_test.go`
- `usage-service/internal/httpapi/server.go`
- `src/services/api/usageService.ts`
- `src/features/monitoring/hooks/useUsageData.ts`
- `.osc/tasks/05-22-fix-monitoring-usage-performance/**`

## Detected Gates:

- Gate Name: Frontend type-check
  - Confidence: High
  - Evidence: `.github/workflows/pr-check.yml:36` runs `npm run type-check`; `package.json:13` defines `type-check`.
- Gate Name: Frontend lint
  - Confidence: High
  - Evidence: `.github/workflows/pr-check.yml:39` runs `npm run lint`; `package.json:11` defines `lint`.
- Gate Name: Frontend build
  - Confidence: High
  - Evidence: `.github/workflows/pr-check.yml:42` runs `npm run build` with `VERSION=ci`; `package.json:8` defines `build`.
- Gate Name: Frontend tests
  - Confidence: Medium
  - Evidence: `package.json:10` defines `npm test`; PR workflow does not run it.
- Gate Name: Usage Service tests
  - Confidence: High
  - Evidence: `.github/workflows/pr-check.yml:61` runs `go test ./...` from `usage-service/`.
- Gate Name: Docker build
  - Confidence: High
  - Evidence: `.github/workflows/pr-check.yml:79` builds `Dockerfile.usage-service` with `push: false`; `.github/workflows/ghcr-image.yml:72` builds and pushes the same Dockerfile.
- Gate Name: Diff hygiene
  - Confidence: Medium
  - Evidence: repository uses Git patch workflow; local `git diff --check` verifies whitespace errors.

## Suggested Gate Run (Local):

1. `go test ./internal/store` from `usage-service/`
   - Rationale: focused coverage for the new recent-event index and raw JSON behavior.
2. `go test ./internal/httpapi` from `usage-service/`
   - Rationale: focused coverage around the handler module touched by the endpoint change.
3. `go test ./...` from `usage-service/`
   - Rationale: mirrors the Usage Service PR gate.
4. `npm run type-check`
   - Rationale: mirrors the frontend PR gate.
5. `npm run lint`
   - Rationale: mirrors the frontend PR gate.
6. `VERSION=ci npm run build`
   - Rationale: mirrors the frontend build gate.
7. `npm test`
   - Rationale: local frontend regression suite defined in `package.json`.
8. `docker build -f Dockerfile.usage-service --build-arg VERSION=ci -t cpa-manager-usage-performance-test:local .`
   - Rationale: local equivalent of the Docker build gate.
9. `git diff --check`
   - Rationale: patch hygiene.

## If Failures Are Present:

- No local gate failures were present in the completed run.

## Final Self-Review:

- Security & secrets: no secrets committed; management key is used only in existing request auth and transient in-memory request de-duplication.
- Edge cases & error handling: stale/aborted usage requests are ignored; real request errors still set the monitoring error state.
- Backward compatibility / migrations: schema data is unchanged; the new SQLite index is additive; export still uses full `RecentEvents`.
- API/contract compatibility: `/v0/management/usage` response shape is unchanged.
- Observability: no logging changes; no secret-bearing logs added.
- Config/env changes: no new config or env vars.
- Performance risk: dashboard event reads avoid `raw_json`; index matches recent-event ordering; frontend avoids overlapping usage refreshes.
- Rollback plan: revert this commit and redeploy the previous pinned GHCR image; the additive index can safely remain.

## PR-ready checklist:

- [x] Focused store tests: `go test ./internal/store`
- [x] Focused HTTP API tests: `go test ./internal/httpapi`
- [x] Usage Service tests: `go test ./...`
- [x] Frontend type-check: `npm run type-check`
- [x] Frontend lint: `npm run lint`
- [x] Frontend build: `VERSION=ci npm run build`
- [x] Frontend tests: `npm test`
- [x] Docker build: `docker build -f Dockerfile.usage-service --build-arg VERSION=ci -t cpa-manager-usage-performance-test:local .`
- [x] Diff hygiene: `git diff --check`
