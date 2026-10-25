---
title: 'Verify final: approved fix plan against spine, PRD, and EARS'
date: 2026-10-26
checked:
  - architecture-file-brwsr-2026-10-25/ARCHITECTURE-SPINE.md (working tree)
  - prds/prd-file-brwsr-2026-10-25/prd.md (working tree)
  - docs/product/earsRequirments (working tree)
inputs: [reviews/consolidated-findings.md, reviews/verify-fixes.md]
edits_made: none except this report
---

# Verify final: approved fix plan

**Summary**
- Plan items checked: 31 (24 fixes + 3 PRD nits + 4 deferral listings).
- **Applied 30 · Partial 1 (CF-39) · Missing 0.**
- All 9 High items (CF-01 to CF-09) are still applied.
- New contradictions: 0 High, 0 Medium, 4 Low. None blocks approval.
- Spine frontmatter: `status: draft` (L8).
- **Verdict: no blocker for architecture approval: YES.**

Line numbers refer to the current working-tree files. "Spine" means `ARCHITECTURE-SPINE.md`.

## 1. Item status

### Group 1: CF-10 with NC-1 and NC-6 (option a)

| Item | Status | Evidence (current text) |
| --- | --- | --- |
| CF-10 | Applied | Spine AD-8 L149: "a granted pick also sets `hint: 'none'`". L150: "A `notOpened` result sets `hint: 'folderNotOpened'` and changes nothing else. If a session is open, it keeps its entries, query, filter, and results, and the UI shows the hint above them (PRD FR-2, decision CF-10)". AD-12 L239: the hint "is shown above the results when a session is open, and in the start state otherwise". PRD FR-2 L161: "If a folder is already open when the picker closes without granting access, Find keeps the current session and its results, and shows the same hint above the results. [CF-10, decided 2026-10-26]". |
| NC-1 | Applied | Same as CF-10: PRD FR-2 L161 and spine AD-8 L150 now agree. A residual wording issue in the FR-2 lead sentence is listed as NC-A below. |
| NC-6 | Applied | Spine AD-8 L149 ("a granted pick also sets `hint: 'none'`"); PRD FR-2 L160 ("The hint stays until the user chooses a folder successfully, which clears it"). |

### Group 2

| Item | Status | Evidence |
| --- | --- | --- |
| NC-2 | Applied | PRD Glossary L113: "**completed:** every folder was attempted; unreadable folders are counted, not retried". This matches AD-11 rule 9 and FR-12 L237. |
| CF-12 | Applied | AD-22 L363: "The list exposes its size and position (`aria-rowcount`, `aria-rowindex`)". L364: "`primary` changes, notices, *still updating*, and the result of Copy location are announced through one polite `aria-live` region". L365: "A keyboard-only Playwright flow covers: choose a folder (stubbed), search, filter, move to a result, and copy its location". |
| CF-15 | Applied | AD-7 L140: "Playwright tests listing 10,000 and 50,000 stubbed entries assert that no long task exceeds 100 ms". L141: "Newly listed entries are merged into the existing ordered results … The full list is never re-sorted per batch". |
| CF-17 | Applied | AD-20 L338: "unit tests with **coverage of at least 90% on `src/core/**` as a failing gate** (NFR-8)". |
| CF-18 | Applied | S4 L378: "Downloads and Desktop themselves are refused, and the result matches the current Chromium blocklist (re-checked at spike time). Drag-and-drop is out of scope." The spike runs "on Windows, macOS, and Linux" (L373). PRD OQ-1 L355 agrees. |
| CF-21 | Applied | Design Paradigm L36, Adapters row: "`src/core/ports/` only (it re-exports every type an adapter needs)". AD-2 L64: "Adapters import only `src/core/ports/` …". |
| O-1 | Applied | AD-5 L116: "In that fallback, the proxy calls `FolderPicker.pick()` on the main thread, inside the user's click, and passes the granted handle to the worker. A worker can't open the picker. [O-1]". |

### Group 3

| Item | Status | Evidence |
| --- | --- | --- |
| NC-3 | Applied | AD-11 rule 3 L204: "`completed` with 0 entries and `foldersUnreadable === 0` → `emptyFolder`". PRD FR-12 L235: "… and no unreadable folders, Find shows 'This folder has no files' … If some folders couldn't be read, the unreadable-folders notice is shown instead". |
| NC-4 | Applied | AD-11 rule 9 L210: "its text has no count; the count is shown only in the unreadable notice". PRD FR-12 L237: "No matching files found in the folders Find could read (the count appears only in the notice)". |
| NC-5 | Applied | AD-11 rule 7 L208 covers an empty query with an active filter. L221: the notice "says 'files' rather than 'matches'". PRD FR-12 L234: "Filters hide 40 files" and "If no entry has passed the filter yet while listing runs, Find shows 'No matches yet. Still searching…'". See NC-B for a wording issue. |
| CF-14 | Applied | AD-12 L237: "A Playwright test asserts that the anchored row doesn't move while a paused fake listing resumes". |
| CF-22 | Applied | AD-9 L185: "Listeners never call a command synchronously from inside a notification. In development and test builds, every published snapshot, its `results` array, and each `FileEntry` are frozen (`Object.freeze`)". |
| CF-30 | Applied | AD-4 L98–106 has an error-mapping table: picker `AbortError` → `notOpened`; root `NotAllowedError`/`NotFoundError`/`SecurityError` → `accessLost`; subfolder iteration throws → `folderUnreadable`; `getFile()` throws → `modified: 'unavailable'`; no `showDirectoryPicker` → `isSupported()` false. |
| CF-31 | Applied | AD-20 L337: "If the S3 fallback adds a worker, its URL must be built the same way (`new URL(..., import.meta.url)`)". |
| CF-34 | Applied | Result order L395: "`Intl.Collator('en', { sensitivity: 'accent', numeric: false })` (case-insensitive, accent-aware)". |
| CF-38 | Applied | AD-15 L285: "never passes `startIn` or `id`, so the browser remembers no folder". AD-16 L301: "The query and the type filter are never put in the URL, the hash, or the browser history". |
| CF-39 | **Partial** | AD-18 L316 now names `_bmad-output/planning-artifacts/architecture/architecture-file-brwsr-2026-10-25/.memlog.md`. But Deferred L481 ("kept in the decision log as the starting point") and AD-5 L116 ("decision D1") still cite "the decision log" without its path, and the ledger fix asked for both AD-18 and Deferred. |
| CF-41 | Applied | AD-20 L339: "Deploys run only from `main`, and only after every CI check has passed". |
| CF-42 | Applied | AD-12 L238: "Rows display `location` exactly as given. The UI never splits, joins, or rebuilds paths". |
| CF-43 | Applied | Matching L394: "One core function, `splitQuery`, splits the query with exactly `/[\s_.\-]+/u` and drops empty pieces". |
| CF-46 | Applied | AD-14 L270: "No code, including the virtual list, injects `<style>` elements or sets `style.cssText`". This is stated as a standing rule rather than on the Deferred virtual-list line (L483), which is stronger. |

### PRD nits

| Item | Status | Evidence |
| --- | --- | --- |
| FR-15 "Deferred from" line | Applied | PRD L250: "*Deferred from the original FR-15 (§11):* the indicator also covering indexing, and keeping the selected file entry". |
| FR-9 edge-case note | Applied | PRD L219: "**Edge cases (EARS-01):** a query made only of separators is an empty query, and a multi-word query matches its words in any order". |
| §11.3 headings renamed | Applied | PRD L533, L573, L609, L626, L650: "**Former §4.2 Folder Tree and File Details** *(heading as last approved)*" and so on. There are no `### 4.x` headings left inside §11.3. |

### Deferred items (spine *Deferred*)

| Item | Status | Evidence |
| --- | --- | --- |
| CF-26 | Applied | L489: "deferred to the first end-to-end story". |
| CF-36 | Applied | L490: "deferred to the Copy location story". |
| CF-37 | Applied | L491: "deferred to architecture finalize". |
| CF-49 | Applied | L492: "deferred to architecture finalize". |

**Counts:** 30 applied · 1 partial (CF-39) · 0 missing.

## 2. High items CF-01 to CF-09

| ID | Still applied | Current evidence |
| --- | --- | --- |
| CF-01 | Yes | AD-9 L165 `'capped'`; AD-13 L248; AD-11 rule 8 and the cap notice (L217); PRD NFR-4 L283; FR-12 L236. |
| CF-02 | Yes | AD-4 L88 `folderUnreadable`; AD-13 L247; AD-11 rule 9 and the unreadable notice (L218); PRD FR-12 L237. The no-match text changed to "…in the folders Find could read" (NC-4), and the requirement still holds. |
| CF-03 | Yes | Stack L405 (TS 6.0.3), L409–411 (ESLint 10.13.0, typescript-eslint 8.72.0, eslint-plugin-boundaries 7.3.0); AD-2 L67; AD-15 L291. |
| CF-04 | Yes | AD-5 L115–116; AD-9 L179 `selectFolder(): void`, L184. |
| CF-05 | Yes | AD-9 L163–181 (one `SessionSnapshot`, `FileType`, `setTypeFilter`); L187 filter semantics; AD-12 L230; AD-11 L214. |
| CF-06 | Yes | AD-14 L268–269 and L275; Stack L407 `assetsInlineLimit: 0`; S7 L380. |
| CF-07 | Yes | AD-8 L152 ("after every `await`", `addEntries(generation, events)`), L151 cancels the batch timer. |
| CF-08 | Yes | AD-8 L149–150. |
| CF-09 | Yes | AD-4 L86–94 and the L98–106 mapping table. |

## 3. New contradictions

Scope: PRD FR-2, FR-12, and FR-15 against spine AD-8, AD-9, AD-11, and AD-12, plus AD-2 against the Design Paradigm table.

### Checked and consistent

- **Hint behaviour:**
  - Set on `notOpened` (AD-8 L150) and on `accessLost` (AD-13 L250; PRD FR-12 L238).
  - Cleared on a granted pick (AD-8 L149; PRD FR-2 L160).
  - Shown above the results when a session is open and in the start state otherwise (AD-12 L239; PRD FR-2 L159 and L161).
- **`primary` precedence, all 10 rules, traced against the PRD:**
  1. `unsupported`: FR-4.
  2. `start`: the start state and access lost (FR-12 L238).
  3. `emptyFolder`, which now requires no unreadable folders: FR-12 L235.
  4. `prompt`: FR-12 L233.
  5. `results`, with the notices still shown: FR-12 L230.
  6. `filtersHideAll` before rules 7–10: FR-12 L231 says the filter message "replaces the EARS-03 and EARS-04 messages".
  7. `stillSearching`: EARS-04 and FR-12 L229 and L234.
  8. `noMatchCapped`: FR-12 L236.
  9. `noMatchPartial`: FR-12 L237.
  10. `noMatchFinal`: EARS-03 and FR-12 L228.

  No ordering conflicts with the PRD, apart from the wording issues NC-B and NC-C.
- **Notice texts:**
  - The PRD defines all three notices: filter (L230 and L234: "matches" or "files"), cap (L236), and unreadable (L237).
  - It also defines every `primary` key's text except `unsupported`. FR-4 gives only the meaning ("desktop Chrome or Edge is required"), not exact wording. This was already the case before this round.
  - AD-11 L221–222 keeps the text in the PRD and the selection in the core.
- **EARS-03 rev. 3 / EARS-04 rev. 1:**
  - They still never overlap. `completed` and `running` are distinct values of the `Listing` enum (AD-9 L165).
  - Under `capped`, rule 8 applies, and FR-12 L236 says EARS-03 never appears on its own.
  - Under `stopped`, rule 2 returns `start`.
  - The EARS file is unchanged.
- **FR-15 against AD-11 and AD-12:**
  - `stillUpdating = listing === 'running'` (L214).
  - Rows are anchored, with a test (L237).
  - *Still updating* is announced (AD-22 L364).
- **AD-2 against the Design Paradigm table:** they match row by row.
  - Core: nothing outside `src/core/`.
  - Adapters: `src/core/ports/` only.
  - `src/app/`: core, adapters, and UI.
  - UI: `src/core/index.ts` only.

  The mermaid diagram also matches. No contradiction.

### NC-A — Low — The FR-2 lead sentence still says the start state is always shown
- **PRD FR-2:** L156 says "When the folder picker closes without granting access, Find returns to the start state", and L159 says "Find shows the start state with a hint". The new L161 then keeps the session if one is open.
- **Spine:** AD-8 and AD-12 follow L161.
- **Why it's Low:** the specific rule (L161) controls. But the lead sentence and the first consequence read as unconditional.
- **Fix:** begin L156 and L159 with "If no folder is open, …".

### NC-B — Low — "If no entry has passed the filter yet" vs AD-11 rule 6
- **PRD FR-12 L234:** with an empty query and an active filter, "If no entry has passed the filter yet while listing runs, Find shows 'No matches yet. Still searching…'".
- **Spine AD-11:** with entries listed but all excluded by the filter, `filtersHidden > 0`, so rule 6 returns `filtersHideAll` before rule 7. Rule 7 is reached only when nothing has been listed yet. Rule 7's annotation at L208 repeats the misleading phrase "nothing has passed the filter yet".
- **Why it's Low:** PRD L231 (the filter message replaces EARS-04 when the visible list is empty) supports the spine. A test written from L234 alone, however, would expect the wrong key.
- **Fix:** in both places, say "if no file entry has been listed yet".

### NC-C — Low — An empty query and a filter, after a listing with only unreadable folders, gives a no-match message
- **Case:** an empty query, an active filter, a `completed` listing, 0 entries, and `foldersUnreadable > 0`.
- **Spine:** rules 3 (unreadable folders exist), 4 (a filter is active), and 5–8 all fail, so rule 9 returns `noMatchPartial`: "No matching files found in the folders Find could read".
- **PRD:** FR-12 L235 says only that "the unreadable-folders notice is shown instead". Without a filter, rule 4 gives `prompt`, which is coherent.
- **Why it's Low:** this is a rare edge case, and it shows a "no match" message for an empty query.
- **Fix:** accept it, or route this case to `prompt`.

### NC-D — Low — EARS-03 now applies to listings with unreadable folders, which show a qualified message
- **Cause:** the NC-2 fix makes a listing with unreadable folders `completed`. EARS-03 rev. 3 ("listing has completed") therefore applies to it.
- **Conflict:** FR-12 L237 and AD-11 rule 9 show "No matching files found in the folders Find could read", not the exact EARS-03 string.
- **Why it's Low:** the EARS-03 acceptance test (no unreadable folders) is unaffected. The qualified text is a deliberate Principle-1 refinement.
- **Fix:** add one clause to FR-12 L237: "this is the EARS-03 message, qualified". EARS tests must assert exact text (caveat 2 from `verify-fixes.md`).

### Out-of-scope nits (not counted)
- AD-3 L74 still says "Picking, listing, yielding, and **time** reach the core only through ports", but the Clock port is not in the MVP (AD-4 L97).
- The spine frontmatter `updated: '2026-10-25'` doesn't reflect the 2026-10-26 edits.

## 4. Spine status
`status: draft` (spine L8). Confirmed.

## Verdict
**No blocker for architecture approval: YES.**
- Every planned item except CF-39 is applied, and CF-39 is Low and not required before approval.
- All 9 High items hold.
- The 4 new contradictions are all Low, wording-level, and don't change any `SessionController` contract or rule order.
