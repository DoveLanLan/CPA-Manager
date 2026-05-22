# Change Summary: Fix Monitoring Usage Performance

- Date: 2026-05-22
- Owner(s): DoveLanLan
- Related: proposal.md, spec.md, tasks.md

## What changed

- Added a SQLite index for recent usage ordering: `timestamp_ms desc, id desc`.
- Split dashboard event reads from export reads so `/v0/management/usage` can omit `raw_json` while export keeps full event snapshots.
- Updated the Usage Service management handler to use the lighter dashboard read path.
- Added frontend request `AbortSignal` support for Usage Service info and usage calls.
- Made monitoring usage refresh single-flight for unchanged connection settings and abort stale requests when settings change or the component unmounts.
- Added store tests for the new index, payload read behavior, and export compatibility.

## Why

The monitoring page was stuck loading on the VPS because a large `/v0/management/usage` read could exceed the frontend timeout, and the five-second auto-refresh could overlap another request before the previous one finished. The change reduces database payload cost and prevents overlapping refreshes without changing the public JSON response shape.

## Notable decisions

- `RecentEvents` remains the full export/import-compatible path.
- `RecentEventsForPayload` uses the same scan logic but selects `null as raw_json` for dashboard payloads.
- The current VPS `USAGE_QUERY_LIMIT=100` mitigation can remain in place until the new image is deployed and measured.
