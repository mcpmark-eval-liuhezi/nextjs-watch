# NextJS Upstream Health — Merge Summary (compiled 2026-10-25)

Canonical upstream: **vercel/next.js** (default branch `canary`)

> This file mirrors the intended `nextjs-summary.md` gist. Gist creation was blocked by the
> access token's permissions (`403 Resource not accessible by personal access token`), so this
> repository file serves as the shareable link.

## Five most recently merged PRs

| PR | Title |
| --- | --- |
| [#99050](https://github.com/vercel/next.js/pull/99050) | perf(turbopack): look up chunk group indices through a keyed cell |
| [#98685](https://github.com/vercel/next.js/pull/98685) | Turbopack: track availability by chunk group |
| [#99750](https://github.com/vercel/next.js/pull/99750) | refactor(turbopack): chunk chunk groups by key |
| [#98734](https://github.com/vercel/next.js/pull/98734) | Turbopack: consume graph-derived chunk groups |
| [#100280](https://github.com/vercel/next.js/pull/100280) | turbo-tasks-backend: stripe SnapshotCoordinator's operation count and poll for the drain |

## Details

### Discussion categories (vercel/next.js)

Discussions are enabled; 10 categories are currently configured: **App Router, Feedback, Help,
Ideas, Polls, RFC, Security, Show and tell, Triage, Turbopack Error Report**. Use **Help** for
questions and **Feedback / Ideas / RFC** for proposals.

### Per-PR notes

**#99050 — perf(turbopack): look up chunk group indices through a keyed cell** (merged 2026-10-24,
@sokra). Small: 4 files, +58/−25. Adds a keyed-cell `ChunkGroupKeyToIndex` map so `get_index_of`
depends only on the key it looks up, not the entire `ChunkGroupInfo` cell. Files:
`turbopack/crates/turbopack-core/src/module_graph/chunk_group_info.rs` (bulk),
`.../src/chunk/availability_info.rs`, `.../src/chunk/chunk_group.rs`, `crates/next-api/src/app.rs`.
No emitted-output change.

**#98685 — Turbopack: track availability by chunk group** (merged 2026-10-24, @sokra). Large:
78 files — core Rust logic ≈ +560/−120 including a new 360-line `available_chunk_groups.rs`;
the rest are renamed snapshot-test outputs and a new `nested_async_availability` fixture.
Replaces retained available-module sets with roaring bitmaps of chunk-group indices.
Benchmarks: −7% to −30% build time, −10.7% peak RSS on dynamic-import-heavy workloads;
behavior unchanged.

**#99750 — refactor(turbopack): chunk chunk groups by key** (merged 2026-10-24, @sokra). Medium:
20 files, +128/−87. Chunking APIs take `ChunkGroupKey` instead of `ChunkGroup` across
`crates/next-api` (`app.rs`, `instrumentation.rs`, `middleware.rs`) and the turbopack-core
chunking contexts, making `ChunkGroupInfo` the single source of a group's contents.

**#98734 — Turbopack: consume graph-derived chunk groups** (merged 2026-10-24, @sokra).
Medium-large: 11 files, +506/−152. Output chunking uses merged groups derived by
`ChunkGroupInfo` instead of hand-assembled ones; server utilities are chunked as the graph's
shared merged group and client references per layout segment/side. Main files:
`crates/next-api/src/app.rs`, `crates/next-api/src/module_graph.rs`,
`app_client_references_chunks.rs`.

**#100280 — turbo-tasks-backend: stripe SnapshotCoordinator's operation count and poll for the
drain** (merged 2026-10-24, @lukesandberg). Focused: 2 files, +384/−154, 13 commits. Stripes the
shared operation counter into cache-padded per-thread counters (it was 55% of emit-step
samples on a 192-CPU machine, costing 4.7–6.5s); snapshot drain moves to polling with backoff;
`OperationGuard` becomes `!Send`. Files:
`turbopack/crates/turbo-tasks-backend/src/backend/snapshot_coordinator.rs`, `.../src/lib.rs`.

### Context

#98734 → #99750 → #98685 → #99050 are parts 3–6 of one six-PR Turbopack availability stack,
landed together on 2026-10-24. All five merges are Rust-side Turbopack/turbo-tasks performance
work; there are no JS-facing API changes.
