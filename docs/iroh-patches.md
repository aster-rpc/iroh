# Aster iroh/noq fork patch inventory

Last audited: 2026-06-07.

This file records the local Aster patch stack carried on the forked iroh
dependency repos under `/Users/emrul/dev/aster`. Keep it current whenever an
upstream iroh/noq release is imported. The goal is to make the next rebase a
mechanical cherry-pick plus review, not archaeology.

## Scope

Repos covered:

- `/Users/emrul/dev/aster/iroh`
- `/Users/emrul/dev/aster/iroh-blobs`
- `/Users/emrul/dev/aster/iroh-docs`
- `/Users/emrul/dev/aster/iroh-gossip`
- `/Users/emrul/dev/aster/noq`

Current `aster-rpc-internal` pins, at the time of this audit:

| Repo | Aster fork commit/tag | Upstream base imported | New upstream target fetched | Stack range to inspect |
|---|---:|---:|---:|---|
| `iroh` | `77a68a5477` / `aster-iroh-v1.0.0-rc.0` | `v1.0.0-rc.0` (`96b4b0e80d`) | `v1.0.0-rc.1` (`ee8b6a3d93`) | `v1.0.0-rc.0..aster-iroh-v1.0.0-rc.0` |
| `iroh-blobs` | `ede454c774` / `aster-iroh-blobs-v0.101.0` | `v0.101.0` (`7ae446bd`) | `v0.102.0` (`099e7cfd`) | `v0.101.0..aster-iroh-blobs-v0.101.0` |
| `iroh-docs` | `be04181e05` / `aster-iroh-docs-v0.99.0` | `v0.99.0` (`db8d73b`) | `v0.100.0` (`bbb1981`) | `v0.99.0..aster-iroh-docs-v0.99.0` |
| `iroh-gossip` | `d7d13582ba` / `aster-iroh-gossip-v0.99.0` | `v0.99.0` (`65b35eb`) | `v0.100.0` (`37dcb89`) | `v0.99.0..aster-iroh-gossip-v0.99.0` |
| `noq` | `617899fe7e` / `aster-noq-v1.0.0-rc.0` | `noq-v1.0.0-rc.0` (`6ee7cf2f8`) | `noq-v1.0.0-rc.1` (`c80da2500`) | `noq-v1.0.0-rc.0..aster-noq-v1.0.0-rc.0` |

Note: after fetching, `iroh-docs` `upstream/main` is `fc89461`, one commit
past `v0.100.0` (`fix(store): migrate redb 2.x tuple tables on open`). Decide
whether the next Aster port should base on the release tag or upstream main.

## rc1 port progress

`noq` is ported and published. Fork `main` mirrors upstream `main`
(`33dba68d3`), and branch `upgrade/noq-v1.0.0-rc.1` plus tag
`aster-noq-v1.0.0-rc.1` are pushed to `aster-rpc/noq`. The upgrade branch is
based on `noq-v1.0.0-rc.1` (`c80da2500`):

| Commit | Purpose |
|---:|---|
| `7e0f3a7d8` | Cherry-pick of `b2982a787`: `read_into(&mut [u8])` / `poll_read_into` support. |
| `6c1137d29` | Cherry-pick of `00cd03857`: poll-based `noq::poll_driver`. |
| `e968659c4` | Runs `cargo fmt` after the rc1 patch port; the upstream rc1 checkout was not rustfmt-clean under the local toolchain after applying patches. |

Validation completed in `/Users/emrul/dev/aster/noq`:

- `cargo fmt --check`
- `cargo test -p noq-proto --all-features`
- `cargo test -p noq --all-features`

`iroh` is ported and published. Fork `main` mirrors upstream `main`
(`ee8b6a3d93`), and branch `upgrade/iroh-v1.0.0-rc.1` plus tags
`aster-iroh-v1.0.0-rc.1` and `aster-iroh-v1.0.0-rc.1-p1` are pushed to
`aster-rpc/iroh`. The upgrade branch is based on `v1.0.0-rc.1`
(`ee8b6a3d93`). The `-p1` tag is docs-only relative to the code tag and is the
preferred local checkout tag when reading this patch ledger from a detached
worktree.

| Commit | Purpose |
|---:|---|
| `00e016455d` | Cherry-pick of `7793d702a5`: add Aster `noq` `[patch.crates-io]` entries. |
| `e1be58459e` | Cherry-pick of `a6c4e1a2fc`: re-export `noq::poll_driver` as `iroh::poll_driver`. |
| `5aa2363baf` | Pin Aster `noq` patches to rev `e968659c47057b18210a4a03dc2366c65fe8f654` instead of `branch = "main"` because fork `main` now mirrors upstream. |
| `8a751a69f7` | Runs `cargo fmt` after the iroh rc1 patch port. |
| `06bd1f5a90` | Updates `Cargo.lock` to resolve `noq`, `noq-proto`, and `noq-udp` from the Aster rc1 fork rev. |
| `62a6fe93de` | Cherry-pick of `a4efd34a28`: ignore local `docs/aster/` internal notes. |
| `ae6c474ee5` | Records satellite rc1 fork ports and the iroh-docs content-ready patch in this patch ledger. |

Validation completed in `/Users/emrul/dev/aster/iroh`:

- `cargo fmt --check`
- `cargo test -p iroh --all-features -- --test-threads=1`

Note: default threaded `cargo test -p iroh --all-features` compiled but hit a
SIGSEGV in the lib test binary. The named reported tests passed individually,
and the full package run passed with `--test-threads=1`; treat this as a
threaded/local-interface upstream test-run issue to re-check before relying on
the default command in CI.

`iroh-blobs` is ported and published. Fork `main` mirrors upstream `main`
(`099e7cfd`), and branch `upgrade/iroh-blobs-v0.102` plus tag
`aster-iroh-blobs-v0.102.0` are pushed to `aster-rpc/iroh-blobs`. The upgrade
branch is based on `v0.102.0` (`099e7cfd`):

| Commit | Purpose |
|---:|---|
| `94964e8e` | Cherry-pick of `14a2286e`: add Aster `iroh` / `noq` `[patch.crates-io]` entries, updated to explicit rc1 revs. |
| `39ad2947` | Ignore local `docs/aster/` internal notes. |
| `c23a66ac` | Updates `Cargo.lock` to resolve Aster `iroh` / `noq` fork crates. |

Validation completed in `/Users/emrul/dev/aster/iroh-blobs`:

- `cargo fmt --check`
- `cargo test --all-features`

`iroh-docs` is ported and published through `aster-iroh-docs-v0.100.0-p2`.
Fork `main` mirrors upstream `main` (`fc89461`), and branch
`upgrade/iroh-docs-v0.100` plus tags `aster-iroh-docs-v0.100.0` and
`aster-iroh-docs-v0.100.0-p1` / `aster-iroh-docs-v0.100.0-p2` are pushed to
`aster-rpc/iroh-docs`. The `2c8ab9b` sync-recovery patch is also published on
`fix/sync-redrive-missing-content-after-resync` so future upstream ports can
cherry-pick the fix branch/range directly. The upgrade branch is based on
`v0.100.0` (`bbb1981`):

| Commit | Purpose |
|---:|---|
| `7efdff0` | Cherry-pick of `81f3461`: add Aster `iroh` / `noq` `[patch.crates-io]` entries, updated to explicit rc1 revs. |
| `87c1c00` | Ignore local `docs/aster/` internal notes. |
| `5acb0f3` | Updates `Cargo.lock` to resolve Aster `iroh` / `noq` fork crates. |
| `4ae3eaf` | Emits `ContentReady` when received doc content is already present locally, with a regression test for the portal-sync retry path. |
| `2c8ab9b` | Re-drives missing entry-content downloads after a successful re-sync, so known entries whose blobs were unavailable during the first sync can recover after the provider comes back. |

Validation completed in `/Users/emrul/dev/aster/iroh-docs`:

- `cargo fmt --check`
- `cargo test sync_emits_content_ready_for_already_local_content -- --nocapture`
- `cargo test --test sync sync_redrives_known_missing_content_after_resync -- --nocapture`
- `N_ENTRIES=2000 RUST_LOG=iroh_docs=info cargo test --test sync sync_gossip_bulk -- --nocapture`
- `cargo test --all-features`
- `cargo clippy --all-features --tests -- -D warnings`

`iroh-gossip` is ported and published. Fork `main` mirrors upstream `main`
(`37dcb89`), and branch `upgrade/iroh-gossip-v0.100` plus tag
`aster-iroh-gossip-v0.100.0` are pushed to `aster-rpc/iroh-gossip`. The upgrade
branch is based on `v0.100.0` (`37dcb89`):

| Commit | Purpose |
|---:|---|
| `511613f` | Cherry-pick of `14a76d5`: add Aster `iroh` / `noq` `[patch.crates-io]` entries, updated to explicit rc1 revs. |
| `0bbff98` | Cherry-pick of `d7d1358`: ignore local `docs/aster/` internal notes. |
| `fdecec8` | Updates `Cargo.lock` to resolve Aster `iroh` / `noq` fork crates. |

Validation completed in `/Users/emrul/dev/aster/iroh-gossip`:

- `cargo fmt --check`
- `cargo test --all-features`

## Branch and tag model

The intended fork layout is:

- `main` on each `aster-rpc/*` fork mirrors upstream `main` plus upstream tags.
  It should not be the long-lived branch that carries Aster-only patches.
- Aster patches live on a branch named `upgrade/<upstream-tag-or-version>`.
  Existing examples are `upgrade/noq-v1.0.0-rc.0`,
  `upgrade/iroh-v1.0.0-rc.0`, `upgrade/iroh-blobs-v0.101`,
  `upgrade/iroh-docs-v0.99`, and `upgrade/iroh-gossip-v0.99`.
- The patched branch tip is tagged with an `aster-*` tag for stable Cargo pins,
  for example `aster-noq-v1.0.0-rc.0` or `aster-iroh-v1.0.0-rc.0`.

So `upgrade/<...>` is a **branch**, not a tag. The stable tag is the matching
`aster-*` tag. Before starting a new upgrade, fetch upstream and tags, make the
fork `main` match upstream `main`, then create/update the `upgrade/<...>` branch
from the upstream release tag and cherry-pick the Aster patches onto it.

Review the upstream changelog and release commits **before** resetting branches
or cherry-picking patches. That review should decide whether upstream already
absorbed an Aster patch, renamed the APIs a patch touches, or changed dependency
constraints enough that a patch is no longer needed.

When publishing to GitHub, use `--force-with-lease` only for the fork `main`
mirror update if required by history shape. Do not rewrite previously published
`upgrade/*` branches or `aster-*` tags unless intentionally replacing a failed
upgrade attempt.

## Reapply order

1. Port `noq` functional patches first.
2. Port `iroh` patches that depend on the `noq` fork.
3. Port `iroh-blobs`, `iroh-docs`, and `iroh-gossip` dependency patch blocks.
4. Reapply optional fork docs/gitignore patches where useful.
5. Update `aster-rpc-internal` root `[patch.crates-io]` revs to the new Aster fork commits.

Prefer `git cherry-pick -x <commit>` for provenance. If a patch no longer
applies cleanly, use the "Why / enabled behavior" and "Conflict notes" columns
below as the source of truth for reimplementation.

## Functional patches

These patches affect code or dependency resolution needed by Aster.

| Order | Repo | Commit | Files touched | Why / enabled behavior | Aster dependency | Conflict notes |
|---:|---|---:|---|---|---|---|
| 1 | `noq` | `b2982a787` | `noq-proto/src/connection/assembler.rs`, `noq-proto/src/connection/streams/recv.rs`, `noq/src/recv_stream.rs` | Adds `read_into(&mut [u8])` and `poll_read_into` so recv data can be copied directly into caller-owned buffers without intermediate `Bytes` handles. | `aster_transport_core::CoreRecvStream::read_exact_into` uses `noq::RecvStream::read_into` for lower-allocation frame reads; FFI and Python/TS native call paths use `read_exact_into`. | High conflict risk on `noq-proto` stream/assembler internals. Preserve ordered-only semantics, cancel safety, EOF as `Ok(None)` at the public async layer, and tests/proptests that prove byte correctness. |
| 2 | `noq` | `00cd03857` | `noq/src/lib.rs`, `noq/src/poll_driver.rs` | Adds `noq::poll_driver`, a synchronous poll-based QUIC driver over `noq-proto` for FFI or foreign runtimes that do not want a Tokio-owned endpoint. | Intended for Go/Java/.NET/C ABI bridges and re-exported through `iroh` so downstream FFI code can depend on one public crate. | Very high conflict risk on every noq release because it touches sans-IO endpoint/connection public APIs. Port after `read_into`; re-check `ConnectionHandle`, `StreamId`, endpoint events, path events, and transmit APIs against the new noq-proto version. |
| 3 | `iroh` | `7793d702a5` | `Cargo.toml` | Adds `[patch.crates-io]` entries for `noq`, `noq-udp`, and `noq-proto` pointing at `https://github.com/aster-rpc/noq`. | Ensures iroh builds against the Aster noq fork containing `read_into` and `poll_driver`. | Low conflict risk. Keep this until upstream noq contains equivalent APIs. Under the new branch model, pin this to an explicit Aster fork rev/tag; do not use floating `main`, because fork `main` mirrors upstream. |
| 4 | `iroh` | `a6c4e1a2fc` | `iroh/src/lib.rs` | Re-exports `noq::poll_driver` as `iroh::poll_driver`. | Lets Aster FFI consumers access the poll driver through the iroh dependency instead of importing noq directly. | Low conflict risk, usually one export line near `pub use endpoint::{Endpoint, RelayMode};`. Requires the `noq` poll driver patch to exist first. |
| 5 | `iroh-blobs` | `14a2286e` | `Cargo.toml` | Adds `[patch.crates-io]` entries for Aster `iroh`, `iroh-base`, `iroh-relay`, and `noq` forks so the crate resolves the same fork stack standalone. | Prevents dependency graph splits when testing or building `iroh-blobs` outside `aster-rpc-internal`. | Low conflict risk. Under the new branch model, pin to explicit Aster fork revs/tags; do not preserve stale `ed25519-dalek 3.0.0-pre.6` comments. |
| 6 | `iroh-docs` | `81f3461` | `Cargo.toml` | Adds the same Aster fork `[patch.crates-io]` block for iroh/noq crates. | Keeps docs sync tests and standalone builds on the same endpoint/base/noq fork stack as Aster. | Low conflict risk. Pin to explicit Aster fork revs/tags; do not preserve stale pre.6 wording. |
| 7 | `iroh-docs` | `4ae3eaf` | `src/engine/live.rs`, `tests/sync.rs` | Emits `ContentReady` immediately when `start_download` sees `BlobStatus::Complete`, instead of silently returning. | `portal-sync` waits for `LiveEvent::ContentReady` to mark remotely advertised CAS content as usable/retryable even when the blob was preloaded through another path. | Medium conflict risk around `LiveActor::start_download`, `on_download_ready`, and content-ready propagation. Preserve the regression where the receiver preloads the blob, imports the doc, receives `InsertRemote { content_status: Complete }`, and still gets explicit `ContentReady`. |
| 8 | `iroh-docs` | `2c8ab9b` | `src/engine/live.rs`, `tests/sync.rs` | After a successful sync round, scans local doc entries matching the download policy and re-queues missing content downloads from the synced peer before emitting `PendingContentReady`. | `portal-sync` can recover from the "entry replicated, content blob missing" state after a peer reconnects or stabilizes; repeated `start_sync` calls are enough to fetch content that was unavailable during the original insert event. | Medium conflict risk around `LiveActor::on_sync_finished`, `start_download`, and `PendingContentReady` ordering. Preserve the regression where an entry arrives with `ContentStatus::Missing`, the provider later stores the blob, a re-sync receives no new entries, and the receiver still gets `ContentReady` plus the blob locally. |
| 9 | `iroh-gossip` | `14a76d5` | `Cargo.toml` | Adds the same Aster fork `[patch.crates-io]` block for iroh/noq crates. | Keeps gossip builds on the Aster fork stack and avoids mixing crates.io iroh-base/noq with forked iroh. | Low conflict risk. Pin to explicit Aster fork revs/tags and re-check comments/direct dependency versions after each upstream release. |

## Bookkeeping and documentation patches

These are useful for fork hygiene but are not required for Aster runtime
behavior.

| Repo | Commit | Files touched | Purpose | Reapply notes |
|---|---:|---|---|---|
| `iroh` | `a4efd34a28` | `.gitignore` | Ignores `docs/aster/` internal review docs. | Optional. Keep if internal audit/roadmap docs remain local and should not be committed. |
| `iroh` | `77a68a5477` | `README.md` | Adds Aster fork overview and contribution table. | Optional but useful for fork context. Refresh contribution rows rather than preserving stale "Pending" statuses. |
| `iroh-blobs` | `ede454c7` | `README.md` | Adds Aster fork overview and contribution table. | Optional. Refresh PR statuses before carrying forward. |
| `iroh-docs` | `be04181` | `README.md` | Adds Aster fork overview and contribution table. | Optional. Refresh PR statuses before carrying forward. |
| `iroh-gossip` | `161cec0` | `README.md` | Adds Aster fork overview and contribution table. | Optional. Refresh PR statuses before carrying forward. |
| `iroh-gossip` | `d7d1358` | `.gitignore` | Ignores `docs/aster/` internal review docs. | Optional. |
| `noq` | `0f797670a`, `f303f21fd`, `485a71caa`, `617899fe7` | `README.md`, `.gitignore`, transient roadmap files | Adds then moves internal FFI/perf/security roadmap docs under ignored `docs/aster/`; net committed effect is fork README context and `docs/aster/` ignored. | Optional. If carried forward, preserve only committed fork context and `.gitignore`; keep internal roadmaps untracked/ignored. |

## Suggested rc1 port commands

Example sequence for the next port. This keeps fork `main` aligned with
upstream `main`, then carries Aster changes on `upgrade/<upstream-tag>` branches.

Before running the commands:

1. Read upstream release notes / changelog for every target tag.
2. Inspect the upstream commit range from the previous imported base to the new
   target, e.g. `git log --oneline v1.0.0-rc.0..v1.0.0-rc.1`.
3. Compare that range against the functional patch table above and decide for
   each patch: keep, drop because upstream absorbed it, or rewrite because the
   touched APIs changed.

```bash
# noq first
cd /Users/emrul/dev/aster/noq
git fetch --all --tags --prune
# Review first:
# git log --oneline noq-v1.0.0-rc.0..noq-v1.0.0-rc.1
# git diff --stat noq-v1.0.0-rc.0..noq-v1.0.0-rc.1
git switch main
git reset --hard upstream/main
git push origin main --force-with-lease
git push origin --tags
git switch -c upgrade/noq-v1.0.0-rc.1 noq-v1.0.0-rc.1
git cherry-pick -x b2982a787 00cd03857
# Optional docs/bookkeeping:
# git cherry-pick -x 485a71caa 617899fe7
git tag -a aster-noq-v1.0.0-rc.1 -m "Aster noq v1.0.0-rc.1 fork"
git push origin upgrade/noq-v1.0.0-rc.1 aster-noq-v1.0.0-rc.1

# iroh next
cd /Users/emrul/dev/aster/iroh
git fetch --all --tags --prune
# Review first:
# git log --oneline v1.0.0-rc.0..v1.0.0-rc.1
# git diff --stat v1.0.0-rc.0..v1.0.0-rc.1
git switch main
git reset --hard upstream/main
git push origin main --force-with-lease
git push origin --tags
git switch -c upgrade/iroh-v1.0.0-rc.1 v1.0.0-rc.1
git cherry-pick -x 7793d702a5 a6c4e1a2fc
# Optional docs/bookkeeping:
# git cherry-pick -x a4efd34a28 77a68a5477
git tag -a aster-iroh-v1.0.0-rc.1 -m "Aster iroh v1.0.0-rc.1 fork"
git push origin upgrade/iroh-v1.0.0-rc.1 aster-iroh-v1.0.0-rc.1

# Satellite crates
cd /Users/emrul/dev/aster/iroh-blobs
git fetch --all --tags --prune
# Review first:
# git log --oneline v0.101.0..v0.102.0
# git diff --stat v0.101.0..v0.102.0
git switch main
git reset --hard upstream/main
git push origin main --force-with-lease
git push origin --tags
git switch -c upgrade/iroh-blobs-v0.102 v0.102.0
git cherry-pick -x 14a2286e
git tag -a aster-iroh-blobs-v0.102.0 -m "Aster iroh-blobs v0.102.0 fork"
git push origin upgrade/iroh-blobs-v0.102 aster-iroh-blobs-v0.102.0

cd /Users/emrul/dev/aster/iroh-docs
git fetch --all --tags --prune
# Review first:
# git log --oneline v0.99.0..v0.100.0
# git diff --stat v0.99.0..v0.100.0
git switch main
git reset --hard upstream/main
git push origin main --force-with-lease
git push origin --tags
git switch -c upgrade/iroh-docs-v0.100 v0.100.0
git cherry-pick -x 81f3461
git cherry-pick -x 4ae3eaf
git tag -a aster-iroh-docs-v0.100.0-p1 -m "Aster iroh-docs v0.100.0 content-ready patch"
git push origin upgrade/iroh-docs-v0.100 aster-iroh-docs-v0.100.0-p1

cd /Users/emrul/dev/aster/iroh-gossip
git fetch --all --tags --prune
# Review first:
# git log --oneline v0.99.0..v0.100.0
# git diff --stat v0.99.0..v0.100.0
git switch main
git reset --hard upstream/main
git push origin main --force-with-lease
git push origin --tags
git switch -c upgrade/iroh-gossip-v0.100 v0.100.0
git cherry-pick -x 14a76d5
git tag -a aster-iroh-gossip-v0.100.0 -m "Aster iroh-gossip v0.100.0 fork"
git push origin upgrade/iroh-gossip-v0.100 aster-iroh-gossip-v0.100.0
```

After each successful port:

```bash
cargo fmt --check
cargo test -p noq --all-features              # in noq
cargo test -p iroh --all-features             # in iroh, if practical
cargo test                                    # in each satellite repo, if practical
```

Then update `/Users/emrul/dev/aster/aster-rpc-internal/Cargo.toml`:

```toml
[patch.crates-io]
iroh        = { git = "https://github.com/aster-rpc/iroh",        rev = "<new iroh fork commit>" }
iroh-base   = { git = "https://github.com/aster-rpc/iroh",        rev = "<new iroh fork commit>" }
iroh-relay  = { git = "https://github.com/aster-rpc/iroh",        rev = "<new iroh fork commit>" }
iroh-blobs  = { git = "https://github.com/aster-rpc/iroh-blobs",  rev = "<new iroh-blobs fork commit>" }
iroh-docs   = { git = "https://github.com/aster-rpc/iroh-docs",   rev = "<new iroh-docs fork commit>" }
iroh-gossip = { git = "https://github.com/aster-rpc/iroh-gossip", rev = "<new iroh-gossip fork commit>" }
noq         = { git = "https://github.com/aster-rpc/noq",         rev = "<new noq fork commit>" }
noq-udp     = { git = "https://github.com/aster-rpc/noq",         rev = "<new noq fork commit>" }
noq-proto   = { git = "https://github.com/aster-rpc/noq",         rev = "<new noq fork commit>" }
```

## End-of-cycle local checkout alignment

After publishing tags and updating Aster pins, switch the local fork worktrees
to the final patched tags. This matters for downstream workspaces such as
`portal-sync` that use `[patch.crates-io]` path overrides into
`/Users/emrul/dev/aster`: Cargo resolves whatever those working trees have
checked out, not the git revs pinned in `aster-rpc-internal`. Fork `main`
intentionally mirrors upstream and will not contain Aster-only patches such as
`noq::poll_driver`.

For this rc1 cycle, the expected local checkouts are:

```bash
git -C /Users/emrul/dev/aster/iroh switch --detach aster-iroh-v1.0.0-rc.1-p1
git -C /Users/emrul/dev/aster/noq switch --detach aster-noq-v1.0.0-rc.1
git -C /Users/emrul/dev/aster/iroh-blobs switch --detach aster-iroh-blobs-v0.102.0
git -C /Users/emrul/dev/aster/iroh-docs switch --detach aster-iroh-docs-v0.100.0-p1
git -C /Users/emrul/dev/aster/iroh-gossip switch --detach aster-iroh-gossip-v0.100.0

for repo in iroh noq iroh-blobs iroh-docs iroh-gossip; do
  git -C "/Users/emrul/dev/aster/$repo" status --short --branch
  git -C "/Users/emrul/dev/aster/$repo" tag --points-at HEAD
done
```

Then run at least one path-override consumer check, for example:

```bash
cd /Users/emrul/dev/emrul/portal-sync
cargo check --locked -p portal-cas
```

## Aster validation after updating pins

Run from `/Users/emrul/dev/aster/aster-rpc-internal`:

```bash
cargo fmt --check
cargo clippy --manifest-path bindings/python/rust/Cargo.toml -- -D warnings
cargo test -p aster_transport_core
cargo test -p aster --all-features
uv run maturin develop -m bindings/python/rust/Cargo.toml
uv run pytest tests/python/ -v --timeout=30
```

At minimum, verify frame-read paths that use `CoreRecvStream::read_exact_into`
and any FFI/PyO3 call fast paths. Those are the primary consumers of the noq
`read_into` patch.
