# Spec: Fix Monitoring Usage Performance

- Date: 2026-05-22
- Owner(s): DoveLanLan
- Related: proposal.md, tasks.md

## Repo Snapshot

- Modules/components: React/Vite frontend under `src/`; Go Usage Service under `usage-service/`; Docker image build via `Dockerfile.usage-service`.
- Toolchains: Node 24/npm, TypeScript, Vite, Vitest, Go 1.24.x, SQLite via `modernc.org/sqlite`.
- Existing CI: `.github/workflows/pr-check.yml` runs frontend type-check/lint/build, `go test ./...` in `usage-service`, and Docker build with `Dockerfile.usage-service`.
- Existing release: `.github/workflows/ghcr-image.yml` publishes fork images to GHCR.
- Runtime/deploy: Usage Service serves `management.html` and `/v0/management/usage` from SQLite on port `18317`.
- Confidence: High.
- Evidence: `package.json`, `usage-service/go.mod`, `.github/workflows/pr-check.yml`, `.github/workflows/ghcr-image.yml`, `Dockerfile.usage-service`, `usage-service/internal/store/store.go`, `src/features/monitoring/hooks/useUsageData.ts`.

## Scope

### In scope

- Usage Service SQLite index and recent-event read path for dashboard payloads.
- `/v0/management/usage` handler selection of the optimized read path.
- Monitoring frontend request de-duplication/cancellation for usage refresh.
- Focused tests for store behavior.

### Out of scope

- Export/import response format changes.
- Monitoring page redesign or pagination.
- Changing deployment query limits.

## Acceptance Criteria (testable)

1. The dashboard endpoint builds payloads from events that do not load `raw_json`. (Verify: store test and handler code review)
2. `RecentEvents` still returns `raw_json` for export/import compatibility. (Verify: store test)
3. SQLite init creates an index matching `order by timestamp_ms desc, id desc`. (Verify: store test)
4. `useUsageData.loadUsage` does not start overlapping `/v0/management/usage` calls for the same connection key. (Verify: code review and frontend gates)
5. Existing frontend and Usage Service tests pass. (Verify: CI/local commands)

## Behavior / Requirements

`GET /v0/management/usage` should use a dashboard read path that selects the same logical event fields as before except `raw_json`. The payload shape returned by `usage.BuildPayload` must remain unchanged because the UI does not expose `raw_json`. Export must keep using the full event read path.

Frontend usage loading should return the in-flight request promise when the same API base, Usage Service base, enabled state, and management key are unchanged. If those parameters change, the old request should be aborted and ignored before starting a new one.

## Edge Cases

- Auto-refresh interval fires while a usage request is still pending.
- Management key or Usage Service URL changes while a request is pending.
- Component unmounts while a request is pending.
- Existing SQLite databases need the new index without data rewrites.

## Compatibility Notes

- Backwards compatibility: API response shape is unchanged.
- Data/migrations: schema data remains unchanged; one additional index is created if missing.
- Config/flags: no new runtime config.

## API/UX Decisions

- Inputs/outputs: no public API input or output change.
- States/errors: aborted stale frontend requests do not surface as user-visible errors.
- Telemetry/logging: no logging changes.
- Accessibility/i18n: no UI text changes.
