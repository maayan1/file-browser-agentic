---
title: 'Rubric walk: Find architecture spine'
reviewed: ARCHITECTURE-SPINE.md (status: draft, updated 2026-10-25)
checklist: .claude/skills/bmad-architecture/references/reviewer-gate.md (Good-spine checklist)
driving_spec: prd.md (filename-only MVP; section 11 deferred)
altitude: feature, greenfield, no parent spine
date: 2026-10-25
---

# Rubric walk: Find architecture spine

**Verdict:** The spine's skeleton is sound: a hexagonal core, one writer, and status logic in the core. It is not ready to hand off yet. Two High findings stand: the pinned TypeScript can't run the lint toolchain that half the Rules rely on, and the worker fallback isn't sealed against the UI. Together with the two High findings in `reconcile-inputs.md`, these need fixing first.

**Counts (this review only):** High 2 · Medium 4 · Low 4 (10 findings). Coverage gaps already raised in `reviews/reconcile-inputs.md` (F1 to F20) are cited, not repeated.

Mechanical lint (`reviews/lint.json`): 0 findings.

## Checklist walk

| Checklist item | Result | Notes |
| --- | --- | --- |
| Fixes the real divergence points and misses none | Mostly | Layering, the single writer, IDs, status keys, error levels, matching, and order are all fixed. Missed: the controller's sync/async contract (R2), the snapshot throttle scope (R3), and the snapshot shape (R4). Reconcile F3, F5, F7, and F9 are also open divergence points. |
| Every AD Rule is enforceable and prevents its divergence | Partly | The lint-enforced Rules (AD-2, AD-14 to AD-17) have no toolchain that works with the pinned TypeScript (R1). Some ban lists can't be enforced by name alone (R5). The AD-2 "core types" clause is ambiguous (R6). |
| Nothing under Deferred lets two units diverge | Mostly | Lint tool, CI provider, folder sizes, and monitoring are safe to defer. The virtual-list choice is safe only once reconcile F5 (accessibility) and F7 (scroll anchoring) add rules. Deferring the lint tool is what hides R1. |
| Named tech is verified-current | Yes, with a conflict | `npm view` confirms every pin is the current release on 2026-10-25: typescript 7.0.2, react 19.3.1, vite 8.3.5, vitest 5.0.5, @playwright/test 1.64.1, create-vite 9.3.1. But typescript-eslint's peer range is `typescript >=4.8.4 <6.1.0` (R1). |
| Ratifies a brownfield codebase | N/A | Greenfield, and AD-1 says so correctly. |
| Covers the spec's capabilities | Mostly | Every MVP FR, NFR, and EARS item is in `binds` and the Capability Map. Open gaps are from reconcile: the large-folder notice and cap (F1, High), unreadable subfolders (F2, High), start state, FR-2 hint, and FR-4 (F3, F4), NFR-6 (F5), and Linux (F11). |
| Parent-spine consistency | N/A | No parent spine. |
| Every dimension decided, deferred, or open | Mostly | The envelope is covered: environments, provider, CI, and "no operations" are all in AD-20 and Deferred. Release and rollback are thin (R9). |
| Retired ADs handled cleanly | Yes, minor | AD-6 and AD-21 are tombstoned, their IDs are never reused, and the stack and spikes are cleaned up. AD-5, AD-7, and AD-8 were rewritten in place (R8). Leftovers: Clock and the worker-URL clause (reconcile F13). |

## Findings

### R1 — High — The pinned TypeScript 7.0.2 can't run the lint toolchain the Rules rely on
- **Where:** Stack (TypeScript 7.0.2); AD-2 ("a lint boundary rule fails the build"); AD-14 to AD-17 (lint bans); Deferred ("Lint tooling choice … the build step chooses it").
- **Problem:** `typescript-eslint` (current) declares the peer dependency `typescript >=4.8.4 <6.1.0`. TypeScript 7 is the native port. The obvious ESLint setup won't install cleanly or parse with the pinned compiler, and type-aware bans (R5) depend on it. Because the tool choice is deferred, the first build story will find this out and improvise. One builder may downgrade TypeScript, another may drop type-aware rules, and a third may skip the lint gate. Every Rule enforced by "lint bans" then becomes unenforced.
- **Fix:** Decide now. Either (a) pin TypeScript 6.0.x for type-checking and linting (Vite and Vitest don't need `tsc`), or (b) name a lint and boundary toolchain that is verified against TS 7, such as Oxlint or dependency-cruiser with a stated parser. Then add a spike or CI acceptance check: "the lint gate fails on a deliberate AD-2 violation and on a deliberate `createWritable` call."

### R2 — High — The S3 worker fallback isn't sealed: the UI can come to depend on a synchronous, in-process controller
- **Where:** AD-5 (fallback: "touches `src/app/` and an adapter, never the core's rules"); AD-8 ("Search is synchronous … so no query IDs are needed"); AD-9 (commands and snapshots); S3 fallback ("Only `src/app/` and an adapter change").
- **Problem:** The UI imports `src/core/index.ts` directly, and nothing says commands are fire-and-forget. A UI builder can call `setQuery` and then read state or a return value in the same tick, which works only while the controller is on the main thread. If S3 fails, crossing a worker boundary brings back the stale-result race that AD-8 retired. The UI also has to change, which breaks the fallback's claim.
- **Fix:** Add three things to AD-9:
  - The UI depends on a `SessionController` **interface** exported from `src/core/index.ts`, and `src/app/` supplies the instance.
  - Commands return `void`, and the UI observes state only through `subscribe`.
  - Each snapshot carries `generation` and a monotonic `seq`, and the UI ignores any snapshot older than the latest it has rendered.

  With these, a worker proxy really does stay inside `src/app/` and one adapter.

### R3 — Medium — The ~10/s snapshot throttle doesn't say whether it covers command-triggered snapshots
- **Where:** AD-7 ("Snapshots are published to the UI at most about 10 times per second while listing runs").
- **Problem:** Read literally, a keystroke during listing can wait up to 100 ms for its snapshot. That eats half of the 200 ms NFR-4 budget, and the text field and the results disagree for that time. One controller builder will throttle everything, and another will throttle only listing progress.
- **Fix:** State it outright: listing-progress snapshots are coalesced to at most about 10 per second, and a snapshot caused by a command (`setQuery`, `setTypeFilter`, `clearFilters`, `selectFolder`) is published synchronously, before the command returns.

### R4 — Medium — The snapshot shape is inconsistent and incomplete
- **Where:** AD-9 (`{ status, results, filtersHidden, message, error }`); AD-11 (returns `{ primary, filtersHidden }`); AD-12 (the UI holds only typed text, scroll, and transient feedback).
- **Problems:**
  - AD-9 calls the field `message`, but AD-11 produces `primary`, so builders will name it differently.
  - The snapshot doesn't echo the active type filter. The UI must then keep the checked types itself, which AD-12 forbids, and the FR-3 reset can't propagate (see reconcile F6).
  - The argument shape of `setTypeFilter` (a set of types, or toggling one) isn't fixed.
  - `results` as a fresh immutable array of up to 50,000 entries, ten times a second, has no stated cost bound (see reconcile F8).
- **Fix:** Define one `SessionSnapshot` type in the core: `{ seq, status: { generation, listing }, query, typeFilter: ReadonlySet<Type>, results, primary, filtersHidden, error }` (plus the reconcile F1, F2, and F3 fields). Fix `setTypeFilter(types: ReadonlySet<Type>)`. Say that `results` may share structure between snapshots, and that the UI treats it as read-only.

### R5 — Medium — Some lint bans can't be enforced by name as written
- **Where:** AD-15 (bans `move`, `{ create: true }`, and the content reads `text()`, `arrayBuffer()`, `stream()`, `slice()`); AD-16 (bans `caches` and `navigator.storage`).
- **Problem:** `slice`, `move`, and `text` are everyday names: `Array.prototype.slice`, `String.prototype.slice`, and `Response.text`. A plain name ban either fails the build on correct code, so builders disable it, or gets narrowed differently by each builder. AD-15 also scopes the content-read ban to "the FileSource adapter", which a global ban can't express. And NFR-3 names the origin private file system, which isn't banned (`navigator.storage.getDirectory` is covered only through `navigator.storage`; that's fine, but it should be stated).
- **Fix:** Name the mechanism for each ban:
  - Ban the File System Access write and content-read methods with type-aware rules (method called on `FileSystemFileHandle`, `FileSystemDirectoryHandle`, or `File`), or with `no-restricted-properties` limited to `src/adapters/fs-access/`.
  - Ban `getFile()` outside the FileSource adapter's metadata path.
  - Each ban gets a deliberate-violation fixture test (this ties to R1).

### R6 — Medium — AD-2's "core types" import path for adapters is ambiguous
- **Where:** Design Paradigm table (Adapters may import "`src/core/ports/` and core types"); AD-2.
- **Problem:** "Core types" doesn't name a path. One adapter will import `src/core/index.ts`, another `src/core/model/file-entry.ts`. The boundary rule can't be written precisely, and adapters end up coupled to core internals.
- **Fix:** Adapters import only from `src/core/ports/`. The ports module re-exports every type a port signature uses (`FileEntry`, the `unavailable` marker, error codes). Encode exactly this in the boundary rule.

### R7 — Low — "Decision D1" and "the decision log" are cited without a location
- **Where:** AD-5 ("decision D1", "the memlog keeps it"); AD-18 ("requires a decision-log entry"); Deferred ("kept in the decision log").
- **Problem:** The spine is the build substrate, but the decision log is `.memlog.md` in the spine folder, and nothing says so. AD-18's "decision-log entry" could be read as the memlog, an ADR file, or a PR note, so builders will log in different places.
- **Fix:** Name the path (`architecture-file-brwsr-2026-10-25/.memlog.md`, appended through the memlog script per AGENTS.md) in AD-18 and Deferred. Define D1 inline in one clause.

### R8 — Low — AD-5, AD-7, and AD-8 were rewritten in place under the same IDs
- **Where:** AD-5, AD-7, AD-8; Deferred ("the original AD-5, AD-7, and AD-8 rules"; "Any re-introduction gets new AD IDs").
- **Problem:** The rules' meaning changed: worker to main thread, the protocol retired, query IDs retired. The IDs stayed, with no revision marker, while the spine says re-introductions get new IDs. This is harmless in greenfield, with no stories yet, but a reader of an older memlog entry or review that cites "AD-8" will find a different rule.
- **Fix:** Add "(revised 2026-10-25, scope reduction; the prior rule is in the memlog)" to AD-5, AD-7, and AD-8. Also apply reconcile F13 (Clock's use; tag the worker-URL clause as S3-fallback only).

### R9 — Low — Release and rollback aren't stated in the operational envelope
- **Where:** AD-20; Deferred ("CI provider configuration", "Monitoring and operations").
- **Problem:** Environments, provider, and checks are decided. But nothing says what triggers a deploy (which branch, or a tag), or what happens when the post-deploy smoke test fails. One CI story may deploy on every push to `main`; another may require a tag.
- **Fix:** Add one line to AD-20, for example: "Deploy only from `main` after all CI checks pass. If the post-deploy smoke fails, redeploy the previous commit. There is no other rollback mechanism." Reconcile F11 (Linux missing from S3, S4, and S7) is part of the same envelope.

### R10 — Low — `img-src 'self'` conflicts with Vite's default asset inlining
- **Where:** AD-14 (CSP `img-src 'self'`, `style-src 'self'`); Stack (Vite 8).
- **Problem:** Vite inlines small assets as `data:` URIs by default (`build.assetsInlineLimit`). This includes icons and images referenced from CSS. Those inlined images are blocked by `img-src 'self'`. The AD-20 smoke test checks for 404s, but a CSP-blocked image isn't a 404, so this can ship broken and different UI builders will patch it differently (for example, by loosening the CSP).
- **Fix:** In AD-14, either set `build.assetsInlineLimit: 0`, or allow `data:` for `img-src` only. Extend the smoke test to fail on any `securitypolicyviolation` event during a normal session.

## Retired ADs — detail

- AD-6 and AD-21 have `Binds: none` and "the ID is never reused". The AD-2 exception is withdrawn, `pdfjs-dist` is removed from the Stack, and the Capability Map and Structural Seed no longer mention previews. Clean.
- Retired spikes S1, S2, and S5 are listed with reasons. Clean.
- Leftovers are minor: the Clock port and the "future worker URL" sentence (reconcile F13), and `accessLost` (reconcile F12).
