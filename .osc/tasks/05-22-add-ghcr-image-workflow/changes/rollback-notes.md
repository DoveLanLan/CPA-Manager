# Rollback Notes: Add GHCR Image Workflow

- Date: 2026-05-22
- Related: spec.md, tasks.md

## Rollback strategy

- Delete `.github/workflows/ghcr-image.yml` to stop future GHCR publishes.
- Remove the GHCR notes from `README.md` and `README_CN.md` if the fork no longer publishes images.
- Existing GHCR images remain in the package registry until removed manually from GitHub Packages.

## Data considerations

- No runtime data, SQLite schema, or container volume changes are introduced.

## Deployment rollback

- CLIProxyAPI deployments can remove `CPA_MANAGER_IMAGE` or point it back to `seakee/cpa-manager:latest`.
