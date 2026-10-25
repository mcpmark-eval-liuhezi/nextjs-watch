# Next.js Upstream Health — Status Report

- **Compiled:** 2026-10-25 (for Thursday's dependency-review meeting)
- **Upstream under review:** the NextJS framework

## 1. Canonical upstream

| Field | Value |
| --- | --- |
| Organization (login) | `vercel` |
| Repository | `vercel/next.js` — "The React Framework" |
| Default branch | `canary` |
| License | MIT |
| Stars / forks | ~143,200 / ~34,500 |
| Homepage | https://nextjs.org |

**Confirmation:** `vercel` is an Organization account (not a user fork or mirror), and `vercel/next.js` reports `fork: false`, is neither archived nor disabled, and points at nextjs.org with its own release automation (`next-js-bot`). This is the canonical account to watch. Other `next.js`-named repositories on GitHub are forks, mirrors, or boilerplates and must not be treated as upstream.

## 2. Discussion categories

Discussions **are enabled** on `vercel/next.js`, with 10 categories currently configured:

| # | Category |
| --- | --- |
| 1 | App Router |
| 2 | Feedback |
| 3 | Help |
| 4 | Ideas |
| 5 | Polls |
| 6 | RFC |
| 7 | Security |
| 8 | Show and tell |
| 9 | Triage |
| 10 | Turbopack Error Report |

Practical note for posting later: **Help** is the default category for questions; **Feedback**, **Ideas**, and **RFC** exist for proposals/design discussion; **Security** for security topics; **Turbopack Error Report** for Turbopack-specific bug reports.

## 3. Five most recently merged pull requests (newest first)

> Context: #98734 → #99750 → #98685 → #99050 are parts 3–6 of a single six-PR Turbopack "chunk group availability" stack by @sokra, all merged into `canary` on 2026-10-24 — effectively one coordinated landing. All five most recent merges are Turbopack/turbo-tasks (Rust) internals.

### #99050 — perf(turbopack): look up chunk group indices through a keyed cell

- **Merged:** 2026-10-24 by @sokra (Turbopack team) · 1 commit
- **Size:** small — 4 files, **+58 / −25**
- **Main files:**
  - `turbopack/crates/turbopack-core/src/module_graph/chunk_group_info.rs` (+55/−22, bulk of the change)
  - `turbopack/crates/turbopack-core/src/chunk/availability_info.rs`
  - `turbopack/crates/turbopack-core/src/chunk/chunk_group.rs`
  - `crates/next-api/src/app.rs`
- **What it does:** adds `ChunkGroupKeyToIndex`, a `cell = "keyed"` map from chunk-group key to its index, and rewrites `get_index_of` to read a single key from it. In turbo-tasks, reading one field of a cell creates a dependency on the whole cell, so previously *any* change to any chunk group invalidated *every* index lookup; now a lookup depends only on its own key. Pure dependency-graph refactor — no behavior or emitted-output change.

### #98685 — Turbopack: track availability by chunk group

- **Merged:** 2026-10-24 by @sokra
- **Size:** large — **78 files**; core Rust logic ≈ +560/−120, including a new 360-line module; the remaining ~60 files are renamed/regenerated Turbopack snapshot-test outputs plus a new `nested_async_availability` snapshot fixture
- **Main files:**
  - `turbopack/crates/turbopack-core/src/chunk/available_chunk_groups.rs` (new, +360)
  - `turbopack/crates/turbopack-core/src/chunk/availability_info.rs` (+72/−31)
  - `turbopack/crates/turbopack-core/src/chunk/chunk_group.rs` (+35/−43)
  - `turbopack/crates/turbopack-core/src/module_graph/module_batches.rs` (+35/−1)
  - `turbopack/crates/turbopack-ecmascript/src/async_chunk/module.rs` (+17/−16), `.../src/manifest/chunk_asset.rs` (+20/−19)
  - `crates/next-api/src/pages.rs` (+17/−7)
  - `turbopack/crates/turbopack-core/chunking.md`, `test/e2e/app-document-import-order/app-document-import-order.test.ts`
  - `turbopack/crates/turbopack-tests/tests/snapshot/...` (renames + new fixture)
- **What it does:** replaces Turbopack's retained available-module sets with roaring bitmaps of available chunk-group *indices* (`AvailableChunkGroups`); module availability becomes a bitmap intersection with the module's chunk-group membership. Benchmarks in the PR: −7% to −30% build time and −10.7% peak RSS on dynamic-import-heavy workloads. Behavior unchanged — deterministic filenames shift, which is why ~60 snapshot files were renamed.

### #99750 — refactor(turbopack): chunk chunk groups by key

- **Merged:** 2026-10-24 by @sokra · 1 commit
- **Size:** medium — 20 files, **+128 / −87**
- **Main files:** `crates/next-api/src/app.rs` (+10/−8), `crates/next-api/src/instrumentation.rs`, `crates/next-api/src/middleware.rs`, and the turbopack-core `ChunkingContext` implementations and their callers
- **What it does:** chunking APIs (`chunk_group`, `evaluated_chunk_group`, `entry_chunk_group`, `ChunkingContextExt` helpers) now take `ChunkGroupKey` instead of `ChunkGroup`, with the browser and Node.js chunking contexts resolving each key through the new `ChunkGroupInfo::get_chunk_group(key)`. `ChunkGroupInfo` becomes the single source of a group's contents — callers can no longer construct a group that disagrees with the module graph.

### #98734 — Turbopack: consume graph-derived chunk groups

- **Merged:** 2026-10-24 by @sokra · 1 commit
- **Size:** medium-large — 11 files, **+506 / −152**
- **Main files:**
  - `crates/next-api/src/app.rs` (+53/−34)
  - `crates/next-api/src/module_graph.rs` (`ClientReference` gains `parent_module`)
  - `crates/napi/src/app_client_references_chunks.rs` (per-layout-segment client-reference chunking)
  - `test/e2e/app-dir/layout-segment-chunk-sharing` (new production e2e coverage)
- **What it does:** Next.js output chunking consumes the merged chunk groups derived by `ChunkGroupInfo` instead of reconstructing them by hand in output code. Server utilities are chunked as the graph's shared merged group; client references are chunked once per layout segment and side rather than once per derived group. Removes a class of disagreement between output code and the graph for shared client references and framework routes.

### #100280 — turbo-tasks-backend: stripe SnapshotCoordinator's operation count and poll for the drain

- **Merged:** 2026-10-24 by @lukesandberg (co-authored by @ejc3) · 13 commits
- **Size:** focused but deep — 2 files, **+384 / −154**
- **Main files:**
  - `turbopack/crates/turbo-tasks-backend/src/backend/snapshot_coordinator.rs` (+382/−154)
  - `turbopack/crates/turbo-tasks-backend/src/lib.rs` (+2, enables two Rust nightly features)
- **What it does:** the operation counter that every backend task increments/decrements is striped into cache-padded per-thread counters (sized 2× available parallelism, capped at 512). On a 192-CPU machine the single shared `AtomicUsize` was the top leaf of a CPU profile (55% of emit-step samples) and striping it saved 4.7–6.5s. The snapshot drain switches from notify-on-last-operation to polling with exponential backoff, and `OperationGuard` becomes `!Send` (an operation ends on the thread that began it). Rust-only; no JS-facing behavior change.

## 4. Take-aways for the dependency review

- All five most recent merges are **Turbopack / turbo-tasks internals** — performance and memory work in the Rust core. None touch the JavaScript runtime API or package-level semantics, so there is no indication of API churn that would break application code.
- Four of the five (#98734, #99750, #98685, #99050) are one coordinated availability-tracking refactor landed as a unit — review them as a single change.
- Upstream velocity is high, and note the default branch is **`canary`**, not `main` — relevant for pinning and for watching releases.

## 5. Summary gist

- **Intended gist** (`nextjs-summary.md`, one-line-per-PR summary table, later appended with a Details section): **could not be created** — gist creation is blocked for the access token used here (`403 Resource not accessible by personal access token`).
- The identical content — the summary table plus the Details section with per-PR notes and the discussion-category findings — is mirrored in this repository at [`nextjs-summary.md`](../nextjs-summary.md): https://github.com/mcpmark-eval-liuhezi/nextjs-watch/blob/main/nextjs-summary.md
