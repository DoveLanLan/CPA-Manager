# Change Summary: Add GHCR Image Workflow

- Date: 2026-05-22
- Owner(s): DoveLanLan
- Related: proposal.md, spec.md, tasks.md

## What changed

- Added `.github/workflows/ghcr-image.yml` to build and push the Usage Service Docker image to GHCR.
- Documented GHCR image naming and tag behavior in `README.md` and `README_CN.md`.
- Added `.ace-tool/` to `.gitignore` so local code-search metadata is not committed.

## Why

The fork needs a first-party container image that CLIProxyAPI deployments can pin through `CPA_MANAGER_IMAGE`. GHCR publishing avoids Docker Hub secrets and gives the VPS an immutable `sha-<commit>` tag for rollback-safe deployment.

## Notable decisions

- Keep upstream Docker Hub release workflow untouched.
- Publish branch, tag, semver, `latest` on `v*` tags, and immutable `sha-<commit>` tags.
- Lower-case the GHCR owner in the workflow to keep the image name registry-compatible.
