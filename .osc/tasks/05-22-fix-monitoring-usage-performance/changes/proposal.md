# Proposal: Fix Monitoring Usage Performance

- Date: 2026-05-22
- Owner(s): DoveLanLan
- Stakeholders: CPA-Manager operators, CLIProxyAPI VPS operator
- Status: Accepted

## Context / Problem

The monitoring page depends on `/v0/management/usage`. On the VPS, that endpoint could exceed the frontend timeout when reading larger SQLite history windows, and the frontend 5-second auto-refresh could start another request while the previous one was still pending. The current mitigation caps `USAGE_QUERY_LIMIT=100`, but the fork should fix the underlying query and refresh behavior.

## Goals (Why/What)

- Make dashboard usage reads cheaper by avoiding unnecessary `raw_json` payload reads.
- Add an index that matches the recent-event ordering.
- Avoid overlapping monitoring usage requests from auto-refresh.
- Preserve export/import behavior and existing response shape.

## Constraints

- Keep the `/v0/management/usage` JSON payload compatible with the current UI.
- Do not remove stored `raw_json`; export/import still needs it.
- Keep changes narrow to Usage Service storage/handler and monitoring data loading.

## Non-goals

- No pagination redesign.
- No change to `USAGE_QUERY_LIMIT` defaults in Docker Compose.
- No migration that rewrites existing usage rows.

## Proposed Approach (high-level)

Add a dashboard-specific store method that selects only the columns used by `usage.BuildPayload`, omitting `raw_json`, and use it from the `/v0/management/usage` handler. Keep `RecentEvents` with `raw_json` for export. Add a composite SQLite index on `(timestamp_ms desc, id desc)`. In the frontend hook, make `loadUsage` single-flight for the same connection key and abort stale requests when connection parameters change or the component unmounts.

## Risks & Mitigations

- Risk: A missing field in the dashboard-specific query could remove UI data.
  - Mitigation: Keep the same scan path and add tests for snapshot/model fields and `raw_json` omission only.
- Risk: Aborting requests could hide real errors.
  - Mitigation: Ignore only requests aborted by the hook; other errors still set `error`.

## Open Questions

- None.
