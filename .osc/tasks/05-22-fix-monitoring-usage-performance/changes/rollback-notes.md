# Rollback Notes: Fix Monitoring Usage Performance

- Date: 2026-05-22
- Related: proposal.md, spec.md, tasks.md

## Rollback Path

- Revert the CPA-Manager commit that introduced the Usage Service query/index and frontend single-flight changes.
- Rebuild and deploy the previous known-good GHCR image tag, or point `CPA_MANAGER_IMAGE` back to the earlier pinned image.
- Keep or restore the VPS `USAGE_QUERY_LIMIT=100` mitigation if the old image is redeployed.

## Data and Compatibility

- The new SQLite index is additive and can remain after rollback; it does not rewrite usage rows.
- Stored `raw_json` values are untouched.
- The `/v0/management/usage` response shape is unchanged, so no client migration is required.

## Verification After Rollback

- Confirm `/health` returns healthy status.
- Confirm `management.html#/monitoring` loads with the old image behavior.
- Confirm export still includes full usage event snapshots.
