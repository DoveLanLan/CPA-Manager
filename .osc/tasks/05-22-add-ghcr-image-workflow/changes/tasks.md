# Tasks: Add GHCR Image Workflow

- Date: 2026-05-22
- Owner(s): DoveLanLan
- Related: proposal.md, spec.md

## Assumptions

- The fork owner wants images under `ghcr.io/dovelanlan/cpa-manager`.
- `GITHUB_TOKEN` has package write permission when the workflow declares `packages: write`.

## Checklist

- [x] 1) Add GHCR workflow
  - Target: `.github/workflows/ghcr-image.yml`
  - Change: Build and push `Dockerfile.usage-service` to GHCR on `main`, `v*` tags, and manual dispatch.
  - Verify: File review and YAML parse.

- [x] 2) Document image tags
  - Target: `README.md`, `README_CN.md`
  - Change: Document GHCR image name, `sha-<commit>` production tags, and package visibility/login note.
  - Verify: File review.

- [x] 3) Validate existing gates
  - Target: repository root and `usage-service`
  - Change: Run lint/type-check/build/test and Docker build checks where practical.
  - Verify: `npm run type-check`, `npm run lint`, `npm run build`, `go test ./...`, and `docker build`.

## Notes

- Keep the existing Docker Hub release workflow unchanged.
- Local Docker build was attempted twice but could not fetch Docker Hub base image metadata because of TLS handshake timeouts. Workflow syntax, frontend build, frontend lint/type-check, and Usage Service tests passed locally.
