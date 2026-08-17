# Aster iroh/noq fork patch inventory

Last audited: 2026-08-14.

## 1.0.1 wave (2026-07-02) — current pins

Upstream released `noq-v1.0.1` and iroh `v1.0.1` (which bumps to noq 1.0.1 and
net-tools 0.19.1). `noq-v1.0.1` absorbs the Windows UDP busy-loop root-cause
fix via upstream PR n0-computer/noq#707 (filed from our issue #737): it
disables both `SIO_UDP_CONNRESET` and `SIO_UDP_NETRESET` in
`UdpSocketState::new`, a superset of the Aster hand-rolled `-p1` patch, which
was therefore **dropped** from the noq stack this cycle. The iroh recv-error
log throttle was NOT absorbed upstream and is still carried.

The satellite crates had no new upstream releases; their pins are unchanged
from the 1.0.0 wave.

| Repo | Upstream base | Aster tag | Pinned fork commit | Branch |
|---|---|---|---|---|
| `noq` | `noq-v1.0.1` | `aster-noq-v1.0.1` | `c84091f4c1d62f3c9025e40014fc2db59f239ffc` | `upgrade/noq-v1.0.1` |
| `iroh` | `v1.0.1` | `aster-iroh-v1.0.1-p1` | `b15ae8071384f2643b46600ccfebbc2f4d36b6e9` | `upgrade/iroh-v1.0.1` |
| `iroh-blobs` | `v0.103.0` | `aster-iroh-blobs-v0.103.1-p1` | `dfac111c61946c970c5a98e79112957f94291bc3` | `upgrade/iroh-blobs-v0.103` |
| `iroh-docs` | `v0.101.0` | `aster-iroh-docs-v0.101.3` | `d80d7a6c612b96b5e58e9b888f63f83b74e947e1` | `upgrade/iroh-docs-v0.101` |
| `iroh-gossip` | `v0.101.0` | `aster-iroh-gossip-v0.101.0-p1` | `5c021f998a172b81a69669e71786e14a339fa963` | `upgrade/iroh-gossip-v0.101` |

Port notes:

- noq stack carried: `read_into` + `poll_driver` (cherry-picked clean), fmt
  fixup, plus a new commit `c84091f4c` fixing rust 1.96 clippy lints
  (`manual_let_else`, `needless_range_loop`, `useless_nonzero_new_unchecked`,
  `collapsible_if`, and an `#[allow(clippy::large_enum_variant)]` on
  `PollEvent`) that fire on the patch code under the current toolchain. Future
  ports should cherry-pick the versions of these patches from
  `upgrade/noq-v1.0.1` (which include the lint fixes).
- iroh stack carried: noq fork `[patch.crates-io]` entries (pinned to
  `c84091f4c`), `poll_driver` re-export, gitignore, recv-error log throttle
  (cherry-picked clean from `fix/windows-busy-loop` `022676dabb`), lockfile +
  fmt fixups.
- `iroh-blobs` gained three post-tag commits on 2026-08-13 (functional patch row
  8 below): downloader progress observability, its monotonicity follow-up, and a
  test/contract tightening, released as crate version **0.103.1** and tagged `aster-iroh-blobs-v0.103.1`.
  Upstream base stays `v0.103.0`. The registry's immutable `iroh-blobs 0.103.0`
  was published mid-development and is byte-identical to `1f01a087` (verified by
  diffing the extracted `.crate` against each rev), so consumers already have the
  multifetch progress work. The version bump is hygiene — the delta from that
  archive to the tip is a doc comment plus a test assertion, no behavioural code
  — restoring one-version-one-source-state. Do not yank 0.103.0.
  Caution when reasoning about registry provenance: `publish-native-stack.py
  check` clones the fork at the BOM rev and validates *that*, so it cannot detect
  divergence between the BOM and what the registry actually serves. Extract the
  published `.crate` and diff it. Canonical source branch is
  `feat/multifetch`, based on `v0.103.0`.
- `iroh-docs` is released as **0.101.3** at `d80d7a6c` and tagged
  `aster-iroh-docs-v0.101.3`. It carries four patches over 0.101.1: the
  write-authority retirement facade (row 19), the replica-event backpressure
  fix (row 18), the redrive-test correction (row 20) and a **public API
  addition**, row 21. Upstream base stays `v0.101.0`. **Consume 0.101.3, not
  0.101.2.**

  Row 21 is the only public-surface change in this release: `SyncDetails` is
  re-exported so the `Ok` type of the public `SyncEvent::result` field can be
  named outside the crate, and two swapped doc comments on `SyncEvent` are
  corrected. It adds no behaviour and changes no type — an existing public API
  becomes nameable — but it is a surface change and is recorded as one rather
  than folded into the test patch that needed it.

  0.101.2 at `c1987427` was tagged and pushed to both remotes with a defective
  row 20 (see below). The tag is left in place rather than force-moved — it is
  public, and a moved tag is worse than a superseded one — so 0.101.2 exists as
  a real but superseded release. It was never published to the registry.

  Two of the three are corrections to defects 0.101.1 introduced and shipped,
  both found by taking that release's one known "flaky test" seriously instead
  of quarantining it. Neither was ambient: each bisects to a specific Aster
  patch in this same release line.

  Row 18 closes what portal-sync tracked as OI-015. That was recorded as a
  release-gate flake; it is not one.
  `sync_large_read_import_after_peer_initiated_race` stalled at **exactly 1027
  of 2000** events on three independent runs and then sat until the 60s timeout,
  while a passing run finished in 2.7s. An identical stall count across runs is
  not a race — it is the `bounded(1024)` channel filling. With the channel
  unbounded the test passes 6/6 at ~2.7s. Note the test has been failing since
  it was written and reproduces on the released 0.101.1, so 0.101.1 ships this
  stall.

  A second failure surfaced during that work and is fixed as row 20.
  `sync_redrives_known_missing_content_after_resync` was failing about one run
  in five, and it is a **stale test rather than a defect**: row 16 added a
  second `SyncFinished` and the test, written two months earlier, asserts an
  exact event set with an origin-blind matcher. Bisected to row 16 directly —
  20/20 pass at `6d13844^`, 9 failures in 40 after it — so like row 18 it is a
  regression the 0.101.1 release shipped rather than an ambient flake.

  **Row 20's first implementation was wrong, and 0.101.2 shipped it.** That is
  why 0.101.3 exists. Admitting the resync as an *optional* matcher is not
  enough: `assert_next_unordered_with_optionals` tries required matchers before
  optional ones, so a required `match_sync_finished` broad enough to accept
  `Connect(Resync)` consumes it whenever it arrives first and orphans the real
  sync's event.

  **A second attempt, partitioning completions by origin, was also wrong** and
  never shipped. Excluding `Connect(Resync)` from the required matcher makes it
  unsatisfiable in runs where only a resync completes, converting a 27.5%
  mismatch rate into a 2.2% timeout rate — a rarer failure, not a fixed one.

  **No such partition exists, which is the durable lesson.** Origin identifies
  *why* a synchronisation started; it cannot establish ownership by an
  artificial test phase, nor which completion caused the redrive, and either
  origin may legitimately satisfy phase two. Entry counts do not separate them
  either: a `DirectJoin` and a `Resync` that each reconciled nothing differ in
  origin while sharing an entry-count shape, and a queued reconciliation may
  itself carry entries an earlier snapshot missed. The shipped form therefore
  asserts the *shape* of what must happen and inspects neither field.

  **The measurement that cleared the first attempt was invalid, and that
  matters more than the defect.** It repeated the single test in isolation
  (`cargo test --test sync <name>`), which is close to the worst available
  control: with nothing else on the machine the real sync almost always
  finishes first, so the ordering that triggers the bug barely arises. A
  four-figure pass count gathered under a condition that suppresses the failure
  is not evidence. An earlier revision of this section claimed "120 clean
  settled-binary runs" and called 0.101.2 reliably green; both claims were
  false, and are retracted here rather than deleted.

  **How to validate this test.** Run the whole `sync` suite, not the single
  test — concurrency is the variable that matters — and rely primarily on the
  deterministic unit tests, which pin every ordering in-process without timing.
  Current evidence: 11 unit tests covering DirectJoin-only, Resync-only, both
  orderings, a stale drain, a failed completion, an unrelated event, a
  completion from another peer, and repeated completions in phase 1; plus
  **110 consecutive full-suite runs under concurrency with zero failures**,
  against 11 failures in 40 runs without row 20.

- `iroh-docs` gained the startup reconciliation patch in functional row 16:
  `fix/syncfinish` remains the upstream-based source commit, while
  `6d13844` is its provenance-preserving release cherry-pick. The composed
  release is versioned as **0.101.1** at `6c011608` and tagged
  `aster-iroh-docs-v0.101.1`. An upstream audit on 2026-08-14 found
  `upstream/main` at `ad80e691`; its `PeerState::start_connect` still queues
  only `SyncReason::SyncReport`, so row 16 has not been applied upstream.
  Re-audit that branch before carrying row 16 into the next release line.
- All four iroh-family repos were swept on 2026-08-13 (`-p1` tags) so their own
  `[patch.crates-io]` blocks resolve **Forgejo, at this wave's revisions**. They
  had named the GitHub mirror at the 1.0.0-wave `iroh` (`3c329f0c`) and `noq`
  (`ddb8829d`), with `iroh-docs` additionally on a pre-multifetch `iroh-blobs`.
  Aster's product graph was never affected — its root `[patch.crates-io]`
  overrides these — but each repo's *own* build and test suite compiled a stack
  Aster does not ship, which is precisely where fork patches get validated.
  These blocks reference other forks by revision, so a sweep must run in
  dependency order (`noq` → `iroh` → `iroh-blobs` → `iroh-docs`/`iroh-gossip`)
  and each repo's pin then moves. They touch no `src`, so no crate version
  changes and nothing needs republishing.
- `aster-rpc-internal` bumped `iroh = "=1.0.1"` (exact-pin version spec must
  track the fork crate version) alongside the rev pins; `cargo update` pulled
  transitive `netwatch`/`portmapper` 0.19.1 and `netdev` 0.45 per upstream's
  net-tools bump.

Validation: per-fork fmt/clippy(-D warnings, --all-features --tests)/tests
(noq-proto 385, noq 35, iroh 142 single-threaded), then downstream
`aster_transport_core` (242) + `aster --all-features` (94) Rust tests, PyO3
clippy clean, `maturin develop` + 1117 Python tests.

The 2026-08-14 iroh-docs addition passed its 98-test release suite (3 ignored),
the native-stack source/package audit, Aster package checks and clippy, and a
clean `v0.3.20` candidate run of 612 workspace tests (8 ignored). One aggregate
run hit the pre-existing timing assertion in
`swarm_converges_with_sealed_grant_distribution`; the exact test then passed
six consecutive all-feature runs and the final aggregate run passed.

End-of-cycle local worktree checkouts (for path-override consumers like
`portal-sync`):

```bash
git -C /Users/emrul/dev/aster/noq         switch --detach aster-noq-v1.0.1
# iroh stays on upgrade/iroh-v1.0.1 (this ledger commit is one past the tag)
git -C /Users/emrul/dev/aster/iroh        switch upgrade/iroh-v1.0.1
git -C /Users/emrul/dev/aster/iroh-blobs  switch --detach aster-iroh-blobs-v0.103.1-p1
git -C /Users/emrul/dev/aster/iroh-docs   switch --detach aster-iroh-docs-v0.101.3
git -C /Users/emrul/dev/aster/iroh-gossip switch --detach aster-iroh-gossip-v0.101.0-p1
```

## Historical: Windows UDP busy-loop fix (2026-07-01)

**Resolution (2026-07-02):** the `noq` root-cause fix was absorbed upstream in
`noq-v1.0.1` (PR n0-computer/noq#707, filed from our issue #737, which also
disables `SIO_UDP_NETRESET`) and dropped from the Aster stack. The `iroh`
recv-error log throttle is carried forward as functional patch row 5 below.

Fixes the Aster/iroh endpoint busy-loop (~120% CPU + log flood, `recv error … os
error 10054`) that develops on Windows when a peer/candidate becomes unreachable
mid-session. Root-cause writeup and a deterministic repro live outside the fork at
`/Users/emrul/dev/aster/portal-aster-busyloop-rootcause.md` and
`/Users/emrul/dev/aster/tools/win-connreset-repro`.

Each fix is an isolated, code-only commit on a `fix/windows-busy-loop` branch
based on the **upstream release tag** (so it cherry-picks cleanly onto any future
upstream base), then cherry-picked (`-x`) onto the live `upgrade/*` branch.

| Repo | Fix branch (base) | Fix commit | On `upgrade/*` | What it does |
|---|---|---|---|---|
| `noq` | `fix/windows-busy-loop` (`noq-v1.0.0`) | `2678dcd69` | `d8daa2b18` | `UdpSocketState::new` disables `SIO_UDP_CONNRESET` (`WSAIoctl(…, FALSE)`) so a Windows UDP socket stops failing `recv` with `WSAECONNRESET`/os error 10054 after an ICMP "port unreachable". Root cause. |
| `iroh` | `fix/windows-busy-loop` (`v1.0.0`) | `022676dabb` | `f0d154aca9` | `Transports::poll_recv` throttles the per-transport recv-error `warn!` (first, then 1/256, with a `count=` field; reset on any successful recv) via `should_log_recv_error` + `recv_error_log_tests`. Defence-in-depth. |

Validation:

- Windows 11 (`192.168.1.75`, rustc 1.96.0): deterministic repro returns 10054 on
  every recv by default and `WouldBlock` once `SIO_UDP_CONNRESET` is disabled;
  `cargo check -p noq-udp` compiles natively.
- macOS: `cargo check -p noq-udp --target x86_64-pc-windows-msvc`, `cargo test -p
  iroh --lib recv_error_log_tests`, clippy + fmt clean on both.

To land: publish the `fix/windows-busy-loop` branches, cut new `aster-*` code
tags at the `upgrade/*` tips, bump the `aster-rpc-internal` pins + fork manifest,
then fold these rows into the "Functional patches" table below.

## Historical: 1.0 wave (2026-06-21)

Upstream shipped a coordinated 1.0 release wave. All five forks were re-ported
onto the new upstream tags, validated, tagged, and pinned in
`aster-rpc-internal`. This cycle every `aster-*` tag points at exactly the
commit `aster-rpc-internal` pins (pin == tag == validated code — no `-pN`
docs-only retags). This ledger commit sits on the branch *after* the tag, so
the tag stays code-only.

| Repo | Upstream base | Aster tag | Pinned fork commit | Branch |
|---|---|---|---|---|
| `noq` | `noq-v1.0.0` | `aster-noq-v1.0.0` | `ddb8829dbc1cd401f8643ba9b93c542ffee43577` | `upgrade/noq-v1.0.0` |
| `iroh` | `v1.0.0` | `aster-iroh-v1.0.0` | `3c329f0c350b98a772bccdf697bfcc519fcdf25a` | `upgrade/iroh-v1.0.0` |
| `iroh-blobs` | `v0.103.0` | `aster-iroh-blobs-v0.103.0` | `60d098c99d6a759b5b1bf0df635b149b50c74775` | `upgrade/iroh-blobs-v0.103` |
| `iroh-docs` | `v0.101.0` | `aster-iroh-docs-v0.101.0-p1` | `d9c091511de89a3e1310b23072804118822cebb6` | `upgrade/iroh-docs-v0.101` |
| `iroh-gossip` | `v0.101.0` | `aster-iroh-gossip-v0.101.0-p1` | `5c021f998a172b81a69669e71786e14a339fa963` | `upgrade/iroh-gossip-v0.101` |

Patch stacks carried forward unchanged from the rc.1 cycle (all cherry-picked
clean except `iroh-blobs`/`iroh-docs` Cargo.lock regen and a trailing-EOF
patch-block conflict in `iroh-gossip` Cargo.toml). Forced transitive bumps from
upstream's `feat!: update to 1.0 dependencies`: `ed25519-dalek`
`=3.0.0-pre.7` -> `=3.0.0-rc.0`, and `iroh-mdns-address-lookup` `0.3` -> `0.4`.
No `aster-rpc-internal` core API changes were needed (already on rc.1).
Validated: per-fork fmt/clippy/tests, then `aster_transport_core` (229) +
`aster` (68) Rust tests and `maturin develop` + 1117 Python tests.

End-of-cycle local worktree checkouts (for path-override consumers like
`portal-sync`):

```bash
git -C /Users/emrul/dev/aster/noq         switch --detach aster-noq-v1.0.0
git -C /Users/emrul/dev/aster/iroh        switch --detach aster-iroh-v1.0.0
git -C /Users/emrul/dev/aster/iroh-blobs  switch --detach aster-iroh-blobs-v0.103.0
git -C /Users/emrul/dev/aster/iroh-docs   switch --detach aster-iroh-docs-v0.101.0-p1
git -C /Users/emrul/dev/aster/iroh-gossip switch --detach aster-iroh-gossip-v0.101.0-p1
```

(Note: the `iroh` worktree must stay on `upgrade/iroh-v1.0.0` if you want this
ledger present, since the ledger commit is one past `aster-iroh-v1.0.0`.)

## Historical: rc.1 audit (2026-06-07)

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
| `iroh-blobs` | `e20ff0292d` / `fix/iroh-blobs-batch-status` | `v0.102.0` (`099e7cfd`) | `v0.102.0` (`099e7cfd`) | `v0.102.0..fix/iroh-blobs-batch-status` |
| `iroh-docs` | `de983790c8` / `fix/batched-content-download-scheduling` | `v0.100.0` (`bbb1981`) | `v0.100.0` (`bbb1981`) | `v0.100.0..fix/batched-content-download-scheduling` |
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

`iroh-blobs` is ported and published through `e20ff029`. Fork `main` mirrors
upstream `main` (`099e7cfd`), branch `upgrade/iroh-blobs-v0.102` is pushed to
`aster-rpc/iroh-blobs`, and tag `aster-iroh-blobs-v0.102.0` marks the base rc1
port. The batch read and batch status performance fixes are also published on
`fix/iroh-blobs-batch-small-reads` and `fix/iroh-blobs-batch-status` so future
upstream ports can cherry-pick the fix branches/ranges directly. The upgrade
branch is based on `v0.102.0` (`099e7cfd`):

| Commit | Purpose |
|---:|---|
| `94964e8e` | Cherry-pick of `14a2286e`: add Aster `iroh` / `noq` `[patch.crates-io]` entries, updated to explicit rc1 revs. |
| `39ad2947` | Ignore local `docs/aster/` internal notes. |
| `c23a66ac` | Updates `Cargo.lock` to resolve Aster `iroh` / `noq` fork crates. |
| `a0cbdd15` | Adds `Blobs::get_bytes_many_if_complete`, with a batched fs-store inline read path and existing export fallback for non-inline complete blobs. |
| `e20ff029` | Adds `Blobs::status_many`, with batched fs-store, mem-store, and readonly-mem status handling for complete/partial/missing blobs. |

Validation completed in `/Users/emrul/dev/aster/iroh-blobs`:

- `cargo fmt --check`
- `cargo test -p iroh-blobs test_get_bytes_many_if_complete -- --nocapture`
- `cargo test -p iroh-blobs test_status_many -- --nocapture`
- `cargo test --all-features`
- `cargo clippy --all-features --tests -- -D warnings`

Downstream validation after wiring through `aster-rpc-internal` and
`portal-cas`:

- `cargo test -p aster_transport_core`
- `cargo test -p aster namespace`
- `cargo test -p portal-cas`
- `cargo test -p portal-sync-session`
- `cargo clippy -p aster_transport_core -p aster -- -D warnings`
- `cargo clippy -p portal-cas -p portal-sync-session -- -D warnings`

Follow-up perf triage on 2026-06-07 tested the hypothesis that H2's remaining
small-file wall clock was caused by per-blob fs-store fsync in iroh-blobs. A
local no-network `import_bao_bytes` probe disproved that for the H2 workload:
500 x 128-byte blobs imported in ~31ms serial / ~11ms concurrent, while 500 x
20KB above-inline blobs imported in ~3.1s serial / ~1.8s concurrent. The H2
files are `fN\n`, below the inline threshold, so no iroh-blobs fs-store patch
was added. The measured fix is downstream in `aster-rpc-internal` commit
`e741e31`, which reuses a node-level blob `Downloader` instead of constructing
one downloader actor and connection pool per hash.

`iroh-docs` is ported and published through `aster-iroh-docs-v0.100.0-p2`.
Fork `main` mirrors upstream `main` (`fc89461`), and branch
`upgrade/iroh-docs-v0.100` plus tags `aster-iroh-docs-v0.100.0` and
`aster-iroh-docs-v0.100.0-p1` / `aster-iroh-docs-v0.100.0-p2` are pushed to
`aster-rpc/iroh-docs`. The `2c8ab9b` sync-recovery patch is also published on
`fix/sync-redrive-missing-content-after-resync` so future upstream ports can
cherry-pick the fix branch/range directly.

The large initial namespace fix is on `fix/large-namespace-initial-sync` at
`14107686520394f1df0ffe08cb45c9ec7a229221`, carrying `e1f9340` and `1410768`
on top of `2c8ab9b`. The bounded redrive follow-up is on
`fix/bounded-sync-redrive` at `027467daa2e29cf18b68e074ab07289b9f45ee22`,
carrying `027467d` on top of that large-namespace stack. The batched content
download scheduling follow-up is on `fix/batched-content-download-scheduling`
at `de983790c850a4406416697d29d948f78b6210fa`, carrying `de98379` on top of
the bounded-redrive stack. `aster-rpc-internal` pins that branch tip in
`Cargo.toml`; the branch is pushed to `aster-rpc/iroh-docs` so Cargo can
resolve the pinned rev from GitHub. The upgrade branch is based on `v0.100.0`
(`bbb1981`):

| Commit | Purpose |
|---:|---|
| `7efdff0` | Cherry-pick of `81f3461`: add Aster `iroh` / `noq` `[patch.crates-io]` entries, updated to explicit rc1 revs. |
| `87c1c00` | Ignore local `docs/aster/` internal notes. |
| `5acb0f3` | Updates `Cargo.lock` to resolve Aster `iroh` / `noq` fork crates. |
| `4ae3eaf` | Emits `ContentReady` when received doc content is already present locally, with a regression test for the portal-sync retry path. |
| `2c8ab9b` | Re-drives missing entry-content downloads after a successful re-sync, so known entries whose blobs were unavailable during the first sync can recover after the provider comes back. |
| `e1f9340` | Adds a collected `Doc::get_many_vec` query path and protocol request so Aster can take full namespace snapshots without depending on the streaming query channel drain path. |
| `1410768` | Makes docs subscription channels non-backpressuring, so a slow event subscriber cannot block the sync actor during large insert bursts. |
| `027467d` | Bounds sync redrive to hashes already known missing for the namespace, avoiding a full namespace entry scan after every successful sync finish. |
| `de98379` | Drains bursts of replica events and batches content-download status checks/scheduling by hash; also fixes queued-hash bookkeeping so concurrent downloads emit `ContentReady` per completed hash while reserving `PendingContentReady` for namespace queue drain. |

Validation completed in `/Users/emrul/dev/aster/iroh-docs`:

- `cargo fmt --check`
- `cargo test sync_emits_content_ready_for_already_local_content -- --nocapture`
- `cargo test --test sync sync_redrives_known_missing_content_after_resync -- --nocapture`
- `N_ENTRIES=2000 cargo test -p iroh-docs sync_large_read_import_after_peer_initiated_race -- --nocapture`
- `N_ENTRIES=2000 RUST_LOG=iroh_docs=info cargo test --test sync sync_gossip_bulk -- --nocapture`
- `cargo test --all-features`
- `cargo clippy --all-features --tests -- -D warnings`
- `cargo test missing_hashes_are_tracked_per_namespace`
- `cargo test sync_redrives_known_missing_content_after_resync`
- `cargo test --test sync test_download_policies -- --nocapture`
- `cargo clippy --all-features --tests -- -D warnings`
- Portal H2 local-path validation:
  `RUSTC_WRAPPER= BASE=500 HIGH=1 PORTAL_SYNC_BLOB_FETCH_TIMEOUT_MS=5000 RUST_LOG=info,portal_sync_session=debug,portal_syncd::sync_supervisor=debug,iroh_docs=info scripts/phase5/sync/h-content/many-small.sh`
  passed with 500 files converged on B after ~22s, sampled content intact, and
  the 501-file probe converged. This validates correctness with the batching
  patches, but does not materially improve the H2 wall-clock versus the prior
  ~22s baseline; remaining time is likely outside this iroh-docs scheduling
  round-trip path.

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
6. Regenerate the fork manifest and the "current pins" section of this ledger
   (see [Updating the manifest after each cycle](#updating-the-manifest-after-each-cycle)).

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
| 5 | `iroh` | `022676dabb` (fix branch) / `4114c449c5` (on `upgrade/iroh-v1.0.1`) | `iroh/src/socket/transports.rs` | `Transports::poll_recv` throttles per-transport recv-error `warn!` logging (first, then 1/256 with a `count=` field; reset on successful recv) via `should_log_recv_error` + `recv_error_log_tests`. | Defence-in-depth against recv-error log floods (originally the Windows `WSAECONNRESET` busy-loop, root-caused upstream in noq 1.0.1); keeps any future recv-error storm from flooding logs. | Low conflict risk. Canonical source is `fix/windows-busy-loop` (based on `v1.0.0`). Check each upstream release for native throttling that would obsolete it. |
| 5 | `iroh-blobs` | `14a2286e` | `Cargo.toml` | Adds `[patch.crates-io]` entries for Aster `iroh`, `iroh-base`, `iroh-relay`, and `noq` forks so the crate resolves the same fork stack standalone. | Prevents dependency graph splits when testing or building `iroh-blobs` outside `aster-rpc-internal`. | Low conflict risk. Under the new branch model, pin to explicit Aster fork revs/tags; do not preserve stale `ed25519-dalek 3.0.0-pre.6` comments. |
| 6 | `iroh-blobs` | `a0cbdd15` | `src/api/blobs.rs`, `src/api/proto.rs`, `src/store/fs.rs`, `src/store/fs/meta.rs`, `src/store/fs/meta/proto.rs`, `src/store/mem.rs`, `src/store/readonly_mem.rs` | Adds `Blobs::get_bytes_many_if_complete`, backed by a batched fs-store inline-data read command plus existing validated export fallback for non-inline complete blobs. | `aster_transport_core::CoreDoc::read_entry_contents_if_complete` exposes this to portal-cas, and `Manifest::read_snapshot` uses it to read all object/link value blobs with two batch calls instead of one status plus one read per entry. | Medium conflict risk around the blob store request enum and fs/meta actor command enums. Preserve response ordering, missing/partial as `None`, inline hash verification, and fallback through `get_bytes` for non-inline complete blobs. |
| 7 | `iroh-blobs` | `e20ff029` | `src/api/blobs.rs`, `src/api/proto.rs`, `src/store/fs.rs`, `src/store/fs/meta.rs`, `src/store/fs/meta/proto.rs`, `src/store/mem.rs`, `src/store/readonly_mem.rs` | Adds `Blobs::status_many`, backed by batched store actor status handling for fs, mem, and readonly-mem stores. | `iroh-docs::LiveActor` uses it to batch local completeness checks while scheduling many content downloads from a RemoteInsert burst. | Medium conflict risk around the blob store request enum and fs/meta actor command enums. Preserve response ordering and exact single-`status` semantics for complete, partial, and missing blobs. |
| 8 | `iroh-blobs` | `a2e15663`..`7e0da6f7` (on `upgrade/iroh-blobs-v0.103`) / `adab99f1`..`02d6a2e3` (`feat/multifetch`) | `src/api/downloader.rs`, `src/util.rs` | Makes download progress observable: root fetch in `Split` mode reports through the real sink instead of `Drain`, minus its legacy `Progress` offsets (`RootProgressFilter`) because split mode runs two independent counters and forwarding both made the offset go backwards, e.g. `[384, 1, 2, …]`; the split fan-in is drained after the futures stream ends (previously a fast final child lost its `PartComplete`); `TryProvider` moves after the `local.is_complete()` early return so an already-resident request is not reported as an attempt; and a new additive `DownloadProgressItem::BytesTransferred(u64)` gives a per-attempt payload total of successfully decoded bytes (`Progress` is a cumulative offset that includes local bytes and is re-based per child, so it cannot be summed). Provider ordering, failover, resumption and split scheduling are unchanged. | `aster_transport_core::CoreBlobsClient::run_download` folds this stream into `CoreFetchReport`, which `aster::Blobs::download_hash_from` / `download_hash_to_store_from` / `download_collection_from` expose to portal-sync for topology-aware provider ranking. | Medium conflict risk in `execute_get` and `handle_download_split_impl`. Canonical source is `feat/multifetch` (based on `v0.103.0`). Preserve: `Progress` monotonicity in split mode; `BytesTransferred` comes from the returned `Stats::payload_bytes_read` on success and from the last raw sink value on failure. The statement order of `pool.get_or_connect` and the local scan is inert (`get_or_connect` is an `async fn`, so the future does nothing until awaited) — it is left as upstream has it and no overlap is claimed. Six tests in `api::downloader::tests` pin each behaviour. `BytesTransferred` is a transfer metric — the sum of what each attempt decoded — not a content size. |
| 9 | `iroh-docs` | `81f3461` | `Cargo.toml` | Adds the same Aster fork `[patch.crates-io]` block for iroh/noq crates. | Keeps docs sync tests and standalone builds on the same endpoint/base/noq fork stack as Aster. | Low conflict risk. Pin to explicit Aster fork revs/tags; do not preserve stale pre.6 wording. |
| 10 | `iroh-docs` | `4ae3eaf` | `src/engine/live.rs`, `tests/sync.rs` | Emits `ContentReady` immediately when `start_download` sees `BlobStatus::Complete`, instead of silently returning. | `portal-sync` waits for `LiveEvent::ContentReady` to mark remotely advertised CAS content as usable/retryable even when the blob was preloaded through another path. | Medium conflict risk around `LiveActor::start_download`, `on_download_ready`, and content-ready propagation. Preserve the regression where the receiver preloads the blob, imports the doc, receives `InsertRemote { content_status: Complete }`, and still gets explicit `ContentReady`. |
| 11 | `iroh-docs` | `2c8ab9b` | `src/engine/live.rs`, `tests/sync.rs` | After a successful sync round, scans local doc entries matching the download policy and re-queues missing content downloads from the synced peer before emitting `PendingContentReady`. | `portal-sync` can recover from the "entry replicated, content blob missing" state after a peer reconnects or stabilizes; repeated `start_sync` calls are enough to fetch content that was unavailable during the original insert event. | Medium conflict risk around `LiveActor::on_sync_finished`, `start_download`, and `PendingContentReady` ordering. Preserve the regression where an entry arrives with `ContentStatus::Missing`, the provider later stores the blob, a re-sync receives no new entries, and the receiver still gets `ContentReady` plus the blob locally. |
| 12 | `iroh-docs` | `e1f9340` | `src/actor.rs`, `src/api.rs`, `src/api/actor.rs`, `src/api/protocol.rs`, `tests/sync.rs` | Adds a collected `get_many_vec` docs query path. This avoids depending on the streaming query channel drain path for large full-namespace snapshots. | Aster core uses `Doc::get_many_vec` for `query_key_exact`, `query_latest_exact`, `query_key_prefix`, and `query_latest_prefix`; portal-sync uses those wrappers during Tree snapshots. | Medium conflict risk around docs query RPC plumbing. Preserve the large read-import, peer-initiated sync regression where a namespace with many content-bearing entries can be queried to completion after sync. |
| 13 | `iroh-docs` | `1410768` | `src/engine.rs` | Makes docs subscription channels non-backpressuring by using unbounded event channels for replica/live subscriptions. This prevents slow subscribers from blocking the sync actor during large insert bursts. | portal-sync owns a dedicated manifest-event watcher, but event fanout still must not backpressure iroh-docs sync/query progress for large namespaces. | Medium conflict risk around subscription/channel plumbing. If upstream changes event fanout, preserve the invariant that a stalled or slow subscriber cannot block docs sync actor progress. |
| 14 | `iroh-docs` | `027467d` | `src/engine/live.rs` | Bounds `redrive_missing_content_for_peer` to hashes already tracked as missing for that namespace, instead of scanning every local entry after each successful sync. | Preserves reconnect recovery from the sync-redrive patch while avoiding O(namespace entries) CPU/blob-status work during portal-sync large namespace progress. | Medium conflict risk around missing-content tracking, download scheduling, and `on_download_ready`. Preserve namespace-scoped missing state, clear it on successful content completion, and emit `ContentReady` for every namespace waiting on the completed hash. |
| 15 | `iroh-docs` | `de98379` | `Cargo.toml`, `Cargo.lock`, `src/engine/live.rs` | Drains up to 1024 ready replica events per live-actor tick, groups content download candidates by hash, uses `Blobs::status_many`, and starts at most one downloader per missing hash. Also separates per-hash `ContentReady` emission from namespace-wide `PendingContentReady` when multiple downloads run concurrently. | Reduces per-entry blob-status actor round-trips during large RemoteInsert bursts without changing portal-cas data layout. | Medium conflict risk around `on_replica_event`, redrive, neighbor content-ready handling, queued hashes, and content-ready ordering. Preserve `test_download_policies`: concurrent downloads in one namespace must emit `ContentReady` for each completed hash, while `PendingContentReady` still waits for the namespace queue to drain. |
| 16 | `iroh-docs` | `cabe333b20` (`fix/syncfinish`) / `6d13844` (on `upgrade/iroh-docs-v0.101`) | `src/engine/state.rs` | Queues one follow-up reconciliation when a `NewNeighbor` sync request races an already-running sync, matching the existing `SyncReport` behavior. | Prevents portal-sync's startup `DirectJoin` from consuming the only reconciliation opportunity before gossip becomes ready, which could leave a post-join publish absent indefinitely. | Low conflict risk in the per-peer sync state machine. Canonical source is `fix/syncfinish`, based directly on `v0.101.0`. Preserve the regression that `NewNeighbor` during `DirectJoin` sets `resync_requested`; keep this separate from any future `Event::Lagged` anti-entropy behavior. |
| 17 | `iroh-gossip` | `14a76d5` | `Cargo.toml` | Adds the same Aster fork `[patch.crates-io]` block for iroh/noq crates. | Keeps gossip builds on the Aster fork stack and avoids mixing crates.io iroh-base/noq with forked iroh. | Low conflict risk. Pin to explicit Aster fork revs/tags and re-check comments/direct dependency versions after each upstream release. |
| 21 | `iroh-docs` | `6fbe973` (integration branch only — see row 20) | `src/engine.rs`, `src/engine/live.rs` | **Public API addition.** Re-exports `SyncDetails`, the `Ok` type of the public `SyncEvent::result` field, which was previously unnameable outside the crate — a caller could read a completion's counts by inference but could not construct or annotate one. Also corrects `SyncEvent::finished`/`started`, whose doc comments were swapped, and documents what origin and entry counts do *not* establish. | Row 20's unit tests construct synthetic completions and cannot be written without a nameable `SyncDetails`. | Low conflict risk. Adds no behaviour and changes no type — it makes an existing public API nameable. If upstream adds the same re-export, this row drops out cleanly. |
| 20 | `iroh-docs` | `6523891` (integration branch only — see note) | `tests/sync.rs` | Replaces the exact-completion multiset in `sync_redrives_known_missing_content_after_resync` with a semantic event collector. Phase 1 requires a neighbour, the missing-content entry, at least one successful completion and a drain, tolerating further completions in any position rather than consuming a single-use slot. Phase 2 is an ordered state machine — completion, then the expected content, then the drain — in which a drain seen before the content belongs to the previous cycle and resets the wait. Failed completions, completions from another peer, and unrelated events remain errors. | Row 16 changed the observable event sequence; this test, written two months earlier, asserted an exact event set with an origin-blind matcher. It failed ~1 run in 5 and 0.101.1 shipped it. | Low conflict risk. **Has no upstream-tag-based topic branch**, deliberately: it needs both row 14's test, an Aster addition, and row 16's behaviour, and those coexist only on the integration branch. If row 16 is ever ported, port this with it. **Do not reintroduce a partition of completions by origin or entry count** — see the release note; no such partition exists, and two attempts at one shipped or nearly shipped defects. |
| 19 | `iroh-docs` | `d47957a` (`fix/capability-downgrade`) / `5976751` (on `upgrade/iroh-docs-v0.101`) | `src/actor.rs`, `src/api.rs`, `src/api/actor.rs`, `src/api/protocol.rs`, `src/store/fs.rs`, `src/sync.rs`, `tests/sync.rs` | Adds `DocsApi::retire_write_authority(doc_id)`, narrowing a namespace's stored capability from Write to Read **without deleting entries**, refusing with an outstanding-reference count while handles remain, and reporting the capability kind now stored rather than the one requested. | Aster exposes this as `Docs::retire_local_write_authority`; portal-sync needs a non-destructive Write-to-Read transition for its grant lifecycle, and neither importing Read (capability merge only upgrades) nor dropping the document (loses entries) can provide one. | Medium conflict risk in the replica open/capability path. Preserve all three properties together: entries survive, the narrowed capability is durable across restart, and a later write import widens again. This is local authority retirement, not cryptographic revocation — a peer holding the namespace secret still writes. |
| 18 | `iroh-docs` | `2761364d` (`fix/replica-event-backpressure`) / `905d4bc` (on `upgrade/iroh-docs-v0.101`) | `src/engine/live.rs` | Makes the live actor's own replica-event channel unbounded. `Subscribers::send` delivers into it with a blocking `tx.send(..).await` executed **inside the sync actor**, so bounding it means a lagging subscriber stops the single actor servicing replica requests. | Completes row 13, which established this exact invariant for the subscription channels and missed this sibling. portal-sync's shape — one large granted Tree syncing beside a small policy namespace — is precisely the burst that overruns the bound. | Low conflict risk (one channel constructor). Preserve the invariant, not the line: bound work per tick via `MAX_REPLICA_EVENT_BATCH`, never the channel's capacity. `sync_large_read_import_after_peer_initiated_race` is the regression; it stalls at exactly 1027 of 2000 events when the channel is bounded. |

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

Note: after the latest performance work, `aster-rpc-internal` pins
`iroh-blobs` to `e20ff0292d` (`fix/iroh-blobs-batch-status`) and `iroh-docs`
to `de983790c8` (`fix/batched-content-download-scheduling`). Until replacement
`aster-*` tags are cut, path-override consumers such as `portal-sync` should
keep those local worktrees on the matching fix/upgrade branch tips when
validating current Aster behavior.

Then run at least one path-override consumer check, for example:

```bash
cd /Users/emrul/dev/emrul/portal-sync
cargo check --locked -p portal-cas
```

## Updating the manifest after each cycle

`docs/aster-fork-manifest.toml` is the machine-readable, consumer-facing list of
what to pin (tags, commits, upstream bases, paste-ready Cargo snippets). It is
**canonical in this repo** (`aster-rpc/iroh`); a synced copy lives at
`aster-rpc-internal/iroh-fork-manifest.toml`. Both must be refreshed every
upgrade cycle, after pins are flipped and validated, so consumers never pin a
stale rev.

After step 5 of [Reapply order](#reapply-order):

1. Edit the canonical `docs/aster-fork-manifest.toml` here: bump `wave`,
   `released`, every `[forks.*]` block (`upstream_base`, `tag`, `commit`,
   `branch`, `crate_version`), the `[transitive]` pins, and the `[snippets]`
   `version_specs` / `patch_crates_io` blocks. The `commit` values must equal
   the revs pinned in `aster-rpc-internal/Cargo.toml` (pin == tag == commit).
2. Update the "current pins" section at the top of this ledger to match.
3. Validate it parses: `python3 -c "import tomllib,sys; tomllib.load(open('docs/aster-fork-manifest.toml','rb'))"`
   (Python ≥ 3.11; otherwise `uv run python -c ...`).
4. Commit on the `upgrade/<iroh-version>` branch (a docs-only commit, kept
   *after* the `aster-*` tag so the tag stays code-only) and push.
5. Sync the copy and commit it in `aster-rpc-internal`:
   `cp docs/aster-fork-manifest.toml /Users/emrul/dev/aster/aster-rpc-internal/iroh-fork-manifest.toml`

Consistency check (run from the `iroh` worktree on the upgrade branch):

```bash
diff <(git -C /Users/emrul/dev/aster/aster-rpc-internal show HEAD:iroh-fork-manifest.toml) \
     docs/aster-fork-manifest.toml && echo "manifests in sync"
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
