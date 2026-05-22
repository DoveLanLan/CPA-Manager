# Regression Checklist: Fix Monitoring Usage Performance

- Date: 2026-05-22
- Related: proposal.md, spec.md, tasks.md

## Gates

- Build: `VERSION=ci npm run build`; Docker build with `Dockerfile.usage-service`.
- Tests: `npm test`; `go test ./...` from `usage-service/`.
- Static checks: `npm run type-check`; `npm run lint`; `git diff --check`.

## Completed

- [x] `go test ./internal/store`
- [x] `go test ./internal/httpapi`
- [x] `go test ./...` from `usage-service/`
- [x] `npm run type-check`
- [x] `npm run lint`
- [x] `VERSION=ci npm run build`
- [x] `npm test`
- [x] `docker build -f Dockerfile.usage-service --build-arg VERSION=ci -t cpa-manager-usage-performance-test:local .`
- [x] `git diff --check`

## Manual Checks After Deploy

- [ ] `GET /health` on the VPS returns healthy status.
- [ ] `GET /v0/management/usage` returns before the frontend timeout.
- [ ] `management.html#/monitoring` renders usage data instead of staying in the loading state.
- [ ] Network panel does not show overlapping `/v0/management/usage` requests during normal auto-refresh.
