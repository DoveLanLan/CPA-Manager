# Proposal: Add GHCR Image Workflow

- Date: 2026-05-22
- Owner(s): DoveLanLan
- Stakeholders: CPA-Manager fork operators, CLIProxyAPI deployment operators
- Status: Accepted

## Context / Problem

The fork needs its own Docker image so VPS deployments can point `CPA_MANAGER_IMAGE` at a fixed fork tag. The upstream repository currently publishes Docker images to Docker Hub from the tag release workflow, which requires Docker Hub secrets and is not suitable as the fork's primary deployment path.

## Goals (Why/What)

- Publish the Usage Service image to GitHub Container Registry under the fork owner.
- Produce immutable `sha-<commit>` tags for production pinning.
- Keep existing Docker Hub release behavior untouched.
- Allow manual image publishing from GitHub Actions.

## Constraints

- Use the existing `Dockerfile.usage-service`.
- Use `GITHUB_TOKEN` and GHCR permissions instead of adding external registry secrets.
- Preserve existing PR checks and release workflow behavior.
- Build both `linux/amd64` and `linux/arm64` images.

## Non-goals

- No Usage Service code or frontend behavior changes.
- No Docker Hub publishing changes.
- No deployment change in this repository.

## Proposed Approach (high-level)

Add a separate `ghcr-image` GitHub Actions workflow triggered by pushes to `main`, `v*` tags, and manual dispatch. It computes the lower-case GHCR image name, logs in to `ghcr.io` with `GITHUB_TOKEN`, generates Docker metadata, and builds/pushes `Dockerfile.usage-service` for `linux/amd64` and `linux/arm64`.

## Risks & Mitigations

- Risk: GHCR package visibility may default to private.
  - Mitigation: Document that operators may need to make the package public or log in on the VPS.
- Risk: Production deployments using floating tags are hard to roll back.
  - Mitigation: Publish and document immutable `sha-<commit>` tags.

## Open Questions

- None.
