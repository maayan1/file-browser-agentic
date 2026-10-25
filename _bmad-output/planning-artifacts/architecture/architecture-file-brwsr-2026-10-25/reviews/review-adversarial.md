---
title: 'Adversarial review: Find architecture spine'
reviewed: ARCHITECTURE-SPINE.md (status: draft, updated 2026-10-25)
lens: adversarial (two units obey every AD to the letter yet build incompatibly)
inputs: [ARCHITECTURE-SPINE.md, prd.md (final), reviews/reconcile-inputs.md]
date: 2026-10-25
---

# Adversarial review: Find architecture spine

**Method:** for each finding, two units one level down (epics or builders) each obey every AD as written, but their builds don't fit together. Each finding names the AD fix that removes the room to diverge. Findings already raised in `reconcile-inputs.md` (F1 to F20) are not repeated. Where a finding touches one of them, the overlap is noted. The spine was not edited.

**Counts:** High 5 · Medium 7 · Low 3 (15 findings)

## High

### A1 — High — The generation guard checks only at yields, so a stale `await` still writes entries
- **Spine:** AD-8 ("Listing work tagged with an older generation is abandoned at its next yield"), AD-7.
- **Build A (listing epic):** checks `generation` only right after `Yielder.yield()`, as AD-8 says. Between yields it does `for await (const e of fileSource.list(handle))` and pushes each entry into the controller's entry store.
- **Build B (controller epic):** on `selectFolder`, increments `generation`, clears the entry store, and starts the new listing. It trusts AD-8 that old work is gone.
- **Clash:** the old listing's pending `await iterator.next()` (or `getFile()`) resolves after the reset and pushes an old-folder entry into the new store before reaching its next yield. A throttled snapshot timer scheduled by the old listing can also fire and publish it. Both builds obey AD-8, and FR-3 still breaks.
- **AD fix (tighten AD-8):** "Every continuation of listing work compares its captured generation with the current one **after every `await`** (iterator step, `getFile()`, yield) and before any write to session state. Writes go only through one controller method `addEntries(generation, entries)` that drops a mismatched generation. Snapshot timers are cancelled on generation change. The old iterator is closed with `return()`." Add a unit test that resolves a stale iterator step after `selectFolder`.

### A2 — High — When `generation` increments is undefined, so a cancelled picker can kill or freeze the session
- **Spine:** AD-8 ("increments a `generation` number each time a folder is selected"), AD-9 (`selectFolder`), AD-13 (cancel is not an error).
- **Build A:** increments `generation` on entry to `selectFolder`, before the picker opens. "Selected" means the command was called.
- **Build B:** increments only after the picker returns a granted handle.
- **Clash:** in Build A, a user who cancels the picker during an active listing abandons it (AD-8). The old results stay with `listing: running` forever, so AD-11 shows `stillSearching` indefinitely, or the UI clears results that Build B keeps. Tests written against one build fail against the other. This is a different question from F3 (what the UI shows after cancel). This one is about the guard's own trigger.
- **AD fix (tighten AD-8):** "`generation` increments only when the picker returns a granted handle, synchronously, in the same task that resets the entries, query, and filter (see F6). A cancelled or blocked picker changes no session state except the FR-2 hint flag."

### A3 — High — The type filter has two owners and no defined command shape
- **Spine:** AD-9 (commands `setTypeFilter`, `clearFilters`; snapshot `{ status, results, filtersHidden, message, error }`), AD-12 (UI holds only typed text, scroll position, transient feedback).
- **Build A (UI epic):** the snapshot doesn't say which types are selected, so it keeps the checkbox state locally and calls `setTypeFilter('pdf')` on each toggle.
- **Build B (core epic):** `setTypeFilter(types: FileType[])` replaces the whole set. `clearFilters()` empties it. An empty array means "a filter that passes nothing".
- **Clash:**
  - A toggle call replaces the set, so selecting PDF then image shows only images (FR-17 broken).
  - **Clear filters** in the filter notice empties the core's set, but the UI's checkboxes stay checked.
  - Unchecking every box in Build A means "no filter", but in Build B it hides everything and shows "Filters hide N matches".
  - The type literals differ, for example `'text'` versus `'textFile'` versus `'text file'`.
- **AD fix (tighten AD-9 and the Errors/Naming conventions):**
  - `type FileType = 'text' | 'pdf' | 'image' | 'other'`, exported from `src/core/index.ts`. The UI maps the literals to labels.
  - `setTypeFilter(types: ReadonlySet<FileType>)` replaces the whole set. An empty set means **no filter active**.
  - The snapshot carries `query: string` and `typeFilter: ReadonlySet<FileType>`. The UI renders checkbox state only from the snapshot and keeps no copy.

### A4 — High — The snapshot shape and the AD-11 output don't line up
- **Spine:** AD-9 (`{ status, results, filtersHidden, message, error }`), AD-11 (status record `{ generation, listing }`; function returns `{ primary, filtersHidden }`), FR-15 (*still updating* indicator).
- **Build A (core):** puts the AD-11 output into `message`, so `message = { primary, filtersHidden }`, and also copies `filtersHidden` to the top level.
- **Build B (UI):** reads `snapshot.message` as a string key, because AD-11 says "UI maps keys to the exact PRD text". It shows the *still updating* indicator only when `message === 'stillSearching'`, so the indicator disappears whenever results are visible during listing. FR-15 requires it while listing runs, with or without results.
- **Clash:** a type mismatch on `message`, two sources for `filtersHidden`, and no field for the FR-15 indicator. `generation` is nested in `status`, so F6's "clear the text on generation change" reads different paths. There is also no rule for whether `error` replaces results or sits beside them.
- **AD fix (rewrite the AD-9 snapshot as one exported type):**
  ```ts
  interface Snapshot {
    generation: number;
    listing: 'idle' | 'running' | 'done';  // plus F1/F3 additions
    stillUpdating: boolean;                // = listing === 'running' (FR-15)
    primary: 'prompt' | 'results' | 'stillSearching' | 'noMatchFinal'; // plus F15 key
    filtersHidden: number;
    results: readonly FileEntry[];
    query: string;
    typeFilter: ReadonlySet<FileType>;
    error: SessionErrorCode | null;
  }
  ```
  Add: "When `error` is non-null, the UI shows only the error view. `primary` and `results` are ignored."

### A5 — High — The FileSource port contract leaves the entry shape and error translation open
- **Spine:** AD-4 (FileSource "yields file entries (name, location, modified date)"), AD-3 (core has no `DOM` lib), AD-13 (entry-level `unavailable`; subfolder failure "recorded once"; session codes).
- **Build A (adapter):** yields `{ name, location, modified?: number }`, leaving `modified` undefined on failure. It lets `DOMException` (`NotFoundError`, `NotAllowedError`) propagate out of the async iterator, and skips unreadable subfolders silently.
- **Build B (core):** expects `modified: number | 'unavailable'`. It catches errors by `e.name`. Without the DOM lib it can't type `DOMException`, so it treats any throw as `accessLost` and stops listing.
- **Clash:**
  - A file whose date can't be read crashes the core's date formatting or is marked by neither side.
  - One unreadable subfolder ends the whole session in Build B, while AD-13 says listing continues.
  - The "recorded once" subfolder failure has no channel through the port. This is the transport behind F2.
- **AD fix (tighten AD-4):** the port is
  ```ts
  type FileSourceEvent =
    | { kind: 'entry'; name: string; location: string; modified: number | 'unavailable' }
    | { kind: 'folderUnreadable'; location: string };
  list(root): AsyncIterable<FileSourceEvent>  // throws only FileSourceError { code: 'accessLost' }
  ```
  - The adapter builds `location` (AD-10) and translates every `DOMException` into an event or `FileSourceError`. No browser error type crosses the port.
  - `type` is derived in the core (F9), never by the adapter.

## Medium

### A6 — Medium — The subscription contract is unspecified, which breaks `useSyncExternalStore` and memoization
- **Spine:** AD-9 ("publishes an immutable snapshot … to subscribers").
- **Build A (core):** `subscribe(listener: (s: Snapshot) => void)`. It mutates the internal sorted `results` array in place for incremental inserts (F8's suggestion) and publishes a new wrapper object around the same array.
- **Build B (UI):** uses `useSyncExternalStore(subscribe, getSnapshot)`, which needs a `getSnapshot()` that returns the same reference until a change. It memoizes the virtual list on `results` identity.
- **Clash:**
  - There is no `getSnapshot`, so the UI adds one that builds a new object on each call, and React loops ("getSnapshot should be cached").
  - With the same array mutated in place, the memoized list never re-renders, and a published "immutable" snapshot changes after publication.
  - Neither build violates AD-9's wording.
- **AD fix (tighten AD-9):**
  - The controller exposes `getSnapshot(): Snapshot` (a stable reference until the next publish) and `subscribe(onChange: () => void): () => void`.
  - Each publish creates a new `results` array, and nothing mutates a published array or entry afterwards. Entries are frozen.
  - Listeners are called after state is consistent and must not call commands synchronously.

### A7 — Medium — The ~10 Hz snapshot cap doesn't say whether it covers command snapshots
- **Spine:** AD-7 ("Snapshots are published to the UI at most about 10 times per second while listing runs. The final listing transition is published immediately"), AD-9 ("After each change, it publishes").
- **Build A (controller):** one throttle covers all publishes, because AD-7 says "at most about 10 per second". A `setQuery` during listing waits up to 100 ms for the next tick.
- **Build B (UI):** debounces typing by 150 ms before calling `setQuery`, to "protect" the main thread.
- **Clash:**
  - Keystroke-to-results latency becomes about 250 ms during listing, past the 200 ms NFR-4 target, although each unit is within its own reading.
  - S3 measures one combination, and the shipped build may be another.
  - AD-9 ("after each change") and AD-7 ("at most 10/s") contradict each other for commands.
- **AD fix (tighten AD-7):** "The rate cap applies only to snapshots caused by newly listed entries, which are coalesced. Snapshots caused by a command (`setQuery`, `setTypeFilter`, `clearFilters`, `selectFolder`) and the listing-done transition are published synchronously at the end of the command. The UI calls `setQuery` on every input event, with no debounce. Any debounce lives in the core constants module, and S3 measures it."

### A8 — Medium — Result order has no final tiebreak, so incremental merge and full sort disagree
- **Spine:** Consistency Conventions → Result order (`Intl.Collator('en', { sensitivity: 'base' })`, name then location).
- **Build A:** a full `Array.prototype.sort` on each change (stable, so ties keep arrival order).
- **Build B:** a binary-search insertion as entries arrive (ties go before or after existing equal items, depending on `<` versus `<=`).
- **Clash:** on a case-sensitive file system, `Report.pdf` and `report.pdf` can sit in the same folder. Their names and locations compare equal under `base`, so the two builds order them differently, the order changes between snapshots, and a fixture asserting full order is flaky. FR-15 ("inserted in result order") can't be tested for ties. This is separate from F16, which is about accent sensitivity.
- **AD fix (tighten the Result order convention):** "Compare by name with the collator, then location with the collator, then by ID by code unit (`a < b`) as the final tiebreak, so the order is total. One exported comparator in the core is used by both sort and insert. The collator options are fixed exactly (no `numeric`)."

### A9 — Medium — No Unicode normalization: macOS names (NFD) never match typed queries (NFC)
- **Spine:** Consistency Conventions → Matching (`toLocaleLowerCase('en')` on both sides), AD-10 (ID is the location exactly as the browser reports it).
- **Build A (adapter):** passes names through as reported. On macOS these are often NFD (`e` followed by U+0301).
- **Build B (core matching):** lowercases the query typed in the UI, which is NFC (`é`, U+00E9), and runs `includes`.
- **Clash:** `résumé` never matches `résumé.pdf` on macOS, breaking EARS-01 and SM-2 for accented names. If either side normalizes the stored name, the ID no longer equals the on-disk name (AD-10), and Copy location may copy text Finder can't match.
- **AD fix (add to the Matching convention):** "The matcher normalizes both query and name to NFC (`normalize('NFC')`) before lowercasing, in the core only. Stored `name` and ID keep the browser's form unchanged (AD-10). Add NFD fixtures to SM-2."

### A10 — Medium — No rule says which directory entries are file entries
- **Spine:** AD-4 (FileSource yields file entries), AD-13, Tunable limits (the 50,000 limit).
- **Build A (adapter):** yields every `kind === 'file'` handle, including `.DS_Store`, `Thumbs.db`, `desktop.ini`, and everything under `.git/` and `node_modules/`.
- **Build B (adapter):** skips dot-files and dot-folders "as noise", and the core's tests assume they're absent.
- **Clash:** result counts, SM-C1 noise, which folders hit the 50,000 limit, and the e2e fixtures all differ by build. Build B also silently removes entries, against Principle 1. The question of whether directories or entries count toward the limit is open too, which also affects F1.
- **AD fix (tighten AD-4):** "FileSource yields every `kind: 'file'` handle in the tree, hidden files included, with no filtering in the adapter. Directories are traversed, not yielded, and don't count toward the limit. Any future exclusion is a core rule with a PRD decision."

### A11 — Medium — The unit test fake and the e2e picker stub have different shapes, and a listing can't be paused
- **Spine:** Consistency Conventions → Tests ("a fake FileSource" and "a stubbed `showDirectoryPicker`"), AD-7, EARS-04 ("while listing is in progress").
- **Build A (core tests):** the fake FileSource yields an array at once. The fake Clock never advances, so the 20 ms slice never trips and listing finishes in one task, which means the EARS-04 "running" state is never observable.
- **Build B (e2e):** the stub fakes `FileSystemDirectoryHandle` with `entries()` only, while the adapter iterates with `values()`. Its fixture tree uses a different JSON format from the unit fixtures.
- **Clash:** the EARS-04 tests are either missing or timing-flaky. The e2e stub doesn't exercise the real adapter path. The SM-2 fixtures are written twice and drift.
- **AD fix (add to the Tests convention):**
  - One fixture format under `tests/fixtures/` (a tree of `{ name, modified | 'unavailable' | throws }`), consumed by both the fake FileSource and the Playwright stub.
  - The stub implements exactly the handle API subset the adapter uses, listed in the adapter's port documentation.
  - The fake FileSource can be gated (paused or released per entry) so tests can hold listing in `running`. Fake Yielder and Clock are controllable.

### A12 — Medium — `filtersHidden` with an empty query is open to two readings
- **Spine:** AD-11 ("the count hidden by filters"), PRD FR-12 ("When the type filter hides matches … with an empty query and an active filter, the result list contains every file entry that passes the filter").
- **Build A:** with an empty query, every entry is a match, so selecting **PDF** shows "Filters hide 9,412 matches" next to the PDF list.
- **Build B:** "matches" requires a query, so `filtersHidden = 0` when the query is empty, and no notice appears.
- **Clash:** different UI states and different EARS-precedence results (AD-11 step 3) for the same input. Unit tests and e2e tests may pick different readings. F14 covers the empty query with zero results, not this count.
- **AD fix (tighten AD-11, after a PM ruling):** "`filtersHidden` = (entries that name-match the query, where an empty query matches all) − (those that also pass the type filter)." Or the alternative ruling, stated explicitly, with a table-driven test row for an empty query plus a filter.

## Low

### A13 — Low — `selectFolder`'s return value is undefined
- **Spine:** AD-9.
- **Build A:** `selectFolder(): Promise<void>` resolves when listing is done. Playwright awaits it.
- **Build B:** it resolves after the grant, or returns `void` and reports only through snapshots.
- **Clash:** e2e tests and UI busy states wait on different events. A UI that awaits Build A's promise blocks a second **Choose folder** click.
- **AD fix:** "Commands return `void`, except `selectFolder(): Promise<'granted' | 'notOpened'>`, which resolves when the picker settles. Progress and completion are observable only through snapshots."

### A14 — Low — Is "location" displayed as the folder path or the full path?
- **Spine:** AD-10 (ID = location, including the file name), PRD UJ-1 ("`Project_Report_Sep.pdf` in `MyFiles/Clients/Acme/`").
- **Build A (UI):** renders `location` (the full path, so the name is repeated).
- **Build B (UI or UX):** renders the parent folder, by stripping the last segment in the UI. That is string logic in the UI, against the spirit of AD-12.
- **AD fix:** "FileEntry carries `location` (the full path, the ID, what Copy location copies) and `folder` (the parent path, computed in the core). The UI displays `folder` and never splits paths."

### A15 — Low — "Spaces" in query splitting is ambiguous
- **Spine:** Consistency Conventions → Matching ("split at spaces, `_`, `-`, and `.`").
- **Build A:** splits on U+0020 only. **Build B:** splits on `/\s/`, which includes tabs, NBSP, and line breaks from a pasted name.
- **Clash:** a pasted `report final` matches in Build B and not in Build A, so SM-2 fixtures disagree.
- **AD fix:** "Split with exactly `/[\s_.\-]+/u` (all Unicode whitespace), drop empty pieces (F14). One exported `splitQuery` in the core, used by matching and by the 'query is empty' check."

## Summary of AD changes proposed

| AD | Change | Findings |
| --- | --- | --- |
| AD-4 | Typed `FileSourceEvent` port, error translation in the adapter, include every file, no filtering | A5, A10 |
| AD-7 | The rate cap covers only listing-driven snapshots; command snapshots are synchronous; no UI debounce | A7 |
| AD-8 | Generation increments on grant only; check after every `await`; one guarded write path; cancel timers | A1, A2 |
| AD-9 | One exported `Snapshot` type; `getSnapshot`/`subscribe`; new arrays per publish; command signatures | A3, A4, A6, A13 |
| AD-11 | `filtersHidden` definition for an empty query | A12 |
| Conventions | Total comparator with an ID tiebreak; NFC in matching; exact split regex; shared fixtures; `FileType` literals; `folder` field | A3, A8, A9, A11, A14, A15 |
