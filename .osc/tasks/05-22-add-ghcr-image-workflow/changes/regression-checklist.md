# Regression Checklist: Add GHCR Image Workflow

- Date: 2026-05-22
- Related: spec.md, tasks.md

## Gates

- Workflow YAML parse: `ruby -e 'require "yaml"; YAML.load_file(".github/workflows/ghcr-image.yml"); puts "workflow yaml ok"'`
- Workflow lint: `GOBIN=/tmp/codex-bin go install github.com/rhysd/actionlint/cmd/actionlint@latest && /tmp/codex-bin/actionlint .github/workflows/ghcr-image.yml`
- Frontend type-check: `npm run type-check`
- Frontend lint: `npm run lint`
- Frontend build: `VERSION=ci npm run build`
- Usage Service tests: `go test ./...` from `usage-service/`
- Diff hygiene: `git diff --check`
- Docker build: `docker build -f Dockerfile.usage-service --build-arg VERSION=ci -t cpa-manager-ghcr-test:local .`

## Results

- Workflow YAML parse passed.
- `actionlint` passed for `.github/workflows/ghcr-image.yml`.
- `npm ci` completed with no reported vulnerabilities.
- `npm run type-check` passed.
- `npm run lint` passed.
- `VERSION=ci npm run build` passed.
- `go test ./...` passed in `usage-service/`.
- `git diff --check` passed.
- Docker build was attempted twice and did not complete because Docker Hub base image metadata fetches timed out during TLS handshake for `node:22-alpine` and `golang:1.24-alpine`.

## Manual Checks

- After pushing to GitHub, confirm the `GHCR Image` workflow completes on `main`.
- Confirm GHCR contains `ghcr.io/dovelanlan/cpa-manager:sha-<commit>` and `ghcr.io/dovelanlan/cpa-manager:main`.
- If the GHCR package remains private, either make it public or run `docker login ghcr.io` on the VPS before deployment.

## Edge-case re-tests

- Run workflow manually from GitHub Actions and confirm it still publishes the immutable SHA tag.
- Tag a test release only if you intentionally want `latest` and semver tags to be published.
