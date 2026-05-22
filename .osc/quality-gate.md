# Quality Gate

- Date: 2026-05-22
- Scope: Add GHCR image workflow for CPA-Manager fork.

## Assumptions:

- The production image should publish under the fork owner, currently `ghcr.io/dovelanlan/cpa-manager`.
- The workflow should preserve existing Docker Hub tag-release behavior.

## Suspected Change Scope:

- `.github/workflows/ghcr-image.yml`
- `README.md`
- `README_CN.md`
- `.gitignore`
- `.osc/tasks/05-22-add-ghcr-image-workflow/**`

## Detected Gates:

- Gate Name: Frontend type-check
  - Confidence: High
  - Evidence: `.github/workflows/pr-check.yml` frontend job runs `npm run type-check`.
- Gate Name: Frontend lint
  - Confidence: High
  - Evidence: `.github/workflows/pr-check.yml` frontend job runs `npm run lint`.
- Gate Name: Frontend build
  - Confidence: High
  - Evidence: `.github/workflows/pr-check.yml` frontend job runs `npm run build` with `VERSION=ci`.
- Gate Name: Usage Service tests
  - Confidence: High
  - Evidence: `.github/workflows/pr-check.yml` usage-service job runs `go test ./...`.
- Gate Name: Docker build
  - Confidence: High
  - Evidence: `.github/workflows/pr-check.yml` docker-build job uses `docker/build-push-action@v6` with `Dockerfile.usage-service` and `push: false`.
- Gate Name: Release Docker publish
  - Confidence: High
  - Evidence: `.github/workflows/release.yml` builds and pushes Docker Hub image only on `v*` tags.

## Suggested Gate Run (Local):

1. `ruby -e 'require "yaml"; YAML.load_file(".github/workflows/ghcr-image.yml"); puts "workflow yaml ok"'`
   - Rationale: validates the new workflow file is parseable YAML.
2. `GOBIN=/tmp/codex-bin go install github.com/rhysd/actionlint/cmd/actionlint@latest && /tmp/codex-bin/actionlint .github/workflows/ghcr-image.yml`
   - Rationale: validates GitHub Actions syntax and expressions.
3. `npm ci`
   - Rationale: installs the frontend dependencies used by the PR workflow.
4. `npm run type-check`
   - Rationale: mirrors the frontend PR gate.
5. `npm run lint`
   - Rationale: mirrors the frontend PR gate.
6. `VERSION=ci npm run build`
   - Rationale: mirrors the frontend build gate.
7. `go test ./...` from `usage-service/`
   - Rationale: mirrors the Usage Service PR gate.
8. `docker build -f Dockerfile.usage-service --build-arg VERSION=ci -t cpa-manager-ghcr-test:local .`
   - Rationale: local equivalent of the Docker build gate.
9. `git diff --check`
   - Rationale: patch hygiene.

## If Failures Are Present:

- Local Docker build failed twice while fetching Docker Hub metadata for `node:22-alpine` and `golang:1.24-alpine`.
- Failure type: local registry/network TLS handshake timeout, before Dockerfile build steps executed.
- Blocking impact: local Docker build could not be completed in this environment; GitHub Actions should still run the same build from GitHub-hosted runners after push.

## Final Self-Review:

- Security & secrets: uses `GITHUB_TOKEN` with `packages: write`; no external registry secrets added.
- Edge cases & error handling: private GHCR package pull requirements are documented.
- Backward compatibility / migrations: no runtime data or schema changes.
- API/contract compatibility: no API changes.
- Observability: workflow labels include Docker metadata source/revision.
- Config/env changes: no runtime env changes.
- Performance risk: CI now performs a multi-arch build on main/tag/manual triggers; existing PR workflow unchanged.
- Rollback plan: delete the GHCR workflow and use existing Docker Hub image or a previous GHCR tag.

## PR-ready checklist:

- [x] Workflow YAML parse: `ruby -e 'require "yaml"; YAML.load_file(".github/workflows/ghcr-image.yml"); puts "workflow yaml ok"'`
- [x] Workflow lint: `GOBIN=/tmp/codex-bin go install github.com/rhysd/actionlint/cmd/actionlint@latest && /tmp/codex-bin/actionlint .github/workflows/ghcr-image.yml`
- [x] Dependency install: `npm ci`
- [x] Frontend type-check: `npm run type-check`
- [x] Frontend lint: `npm run lint`
- [x] Frontend build: `VERSION=ci npm run build`
- [x] Usage Service tests: `go test ./...`
- [x] Diff hygiene: `git diff --check`
- [ ] Docker build: `docker build -f Dockerfile.usage-service --build-arg VERSION=ci -t cpa-manager-ghcr-test:local .` blocked by Docker Hub TLS handshake timeout in this local environment.
