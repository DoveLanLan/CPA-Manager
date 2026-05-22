# Journal 1: Fix Monitoring Usage Performance

- Date: 2026-05-22
- Task: `.osc/tasks/05-22-fix-monitoring-usage-performance`

## Conclusions

- The monitoring page loading issue is likely caused by expensive `/v0/management/usage` reads and overlapping auto-refresh requests.
- The dashboard does not need `raw_json`; export remains the only path that needs full event snapshots.

## Decisions

- Add an additive SQLite index matching `order by timestamp_ms desc, id desc`.
- Use `RecentEventsForPayload` for the dashboard endpoint and keep `RecentEvents` for export/import.
- Make frontend usage refresh single-flight for unchanged connection settings and abort stale requests.

## Verification

- `go test ./internal/store`
- `go test ./internal/httpapi`
- `go test ./...`
- `npm run type-check`
- `npm run lint`
- `VERSION=ci npm run build`
- `npm test`
- `docker build -f Dockerfile.usage-service --build-arg VERSION=ci -t cpa-manager-usage-performance-test:local .`
- `git diff --check`

## Next Steps

- Commit and push the CPA-Manager fork changes.
- Wait for the GHCR image workflow to publish `sha-<commit>`.
- Update the VPS `CPA_MANAGER_IMAGE` pin to the new image and verify the monitoring page.

## Risks and Rollback

- Risk: the frontend still hits a separate API path that blocks rendering. Verify with browser network after deploy.
- Rollback: redeploy the previous pinned GHCR image; the new SQLite index is additive and can safely remain.
