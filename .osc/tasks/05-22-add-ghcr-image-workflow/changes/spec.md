# Spec: Add GHCR Image Workflow

- Date: 2026-05-22
- Owner(s): DoveLanLan
- Related: proposal.md, tasks.md

## Repo Snapshot

- Modules/components: React/Vite frontend under `src/`, Go Usage Service under `usage-service/`, Docker image build via `Dockerfile.usage-service`.
- Toolchains: Node 24/npm for frontend, Go 1.24.x for Usage Service, Docker Buildx for container builds.
- Existing CI: `.github/workflows/pr-check.yml` runs frontend type-check/lint/build, `go test ./...` in `usage-service`, and Docker build with `push: false`.
- Existing release: `.github/workflows/release.yml` builds GitHub release artifacts and pushes `seakee/cpa-manager` to Docker Hub only on `v*` tags.
- Runtime/deploy: `docker-compose.usage.yml` runs `seakee/cpa-manager:latest` on container port `18317` with `/data` volume.
- Confidence: High.
- Evidence: `package.json`, `usage-service/go.mod`, `Dockerfile.usage-service`, `.github/workflows/pr-check.yml`, `.github/workflows/release.yml`, `docker-compose.usage.yml`, `README.md`.

## Scope

### In scope

- New GHCR workflow for Usage Service image publishing.
- Documentation of GHCR image name and tag behavior.

### Out of scope

- Docker Hub release workflow changes.
- Application code changes.
- Deployment compose changes in this repository.

## Acceptance Criteria (testable)

1. Pushes to `main` publish `ghcr.io/<owner>/cpa-manager:sha-<commit>` and `:main`. (Verify: inspect workflow triggers and metadata tags)
2. `v*` tags publish semantic version tags and `:latest`. (Verify: inspect metadata tag rules)
3. Workflow uses `GITHUB_TOKEN` with `packages: write` and does not require Docker Hub secrets. (Verify: inspect workflow permissions and login step)
4. Image build uses `Dockerfile.usage-service` for `linux/amd64` and `linux/arm64`. (Verify: inspect build-push step)
5. Existing Docker Hub release workflow remains untouched. (Verify: `git diff -- .github/workflows/release.yml`)

## Behavior / Requirements

The workflow must compute a lower-case GHCR image name from the repository owner because container image names are expected to be lower-case. It must use Docker metadata labels and tags, cache through GitHub Actions cache, and pass the existing `VERSION` build argument into `Dockerfile.usage-service`.

## Edge Cases

- GHCR image visibility may be private after first publish; VPS pulls then require `docker login ghcr.io`.
- Manual dispatch on a branch publishes the branch tag and the immutable SHA tag.
- Tag publishes still run the existing Docker Hub release workflow separately.

## Compatibility Notes

- Backwards compatibility: Existing Docker Hub image and release workflow continue to work.
- Data/migrations: No data or schema changes.
- Config/flags: No runtime config changes.

## API/UX Decisions

- Inputs/outputs: output image name is `ghcr.io/<lowercase-owner>/cpa-manager`.
- States/errors: GitHub Actions reports build or registry permission failures.
- Telemetry/logging: Docker metadata labels include source and revision.
- Accessibility/i18n: Not applicable.
