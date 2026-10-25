---
title: 'Verify fixes: PRD re-check and spine reviews'
date: 2026-10-25
checked:
  - prds/prd-file-brwsr-2026-10-25/prd.md (working tree)
  - docs/product/earsRequirments (working tree)
  - architecture-file-brwsr-2026-10-25/ARCHITECTURE-SPINE.md (working tree)
  - baseline: prd.md at git commit f0f6934
reviews_verified: [review-scope-reduction-recheck.md, reconcile-inputs.md, review-rubric.md, review-adversarial.md, review-tech-currency.md]
edits_made: none except this report
---

# Verify fixes: PRD re-check and spine reviews

**Summary:**
- **High findings:** 12 resolved, 0 partial, 0 open.
- **PRD re-check mediums:** M-1 to M-4 and M-6 resolved (M-1 and M-6 each leave a small nit). M-5 is partial.
- **§11:** verbatim. All 11 deferred FRs are byte-identical to f0f6934.
- **New contradictions:** 2 medium, 4 low.
- **EARS-03 rev. 3 / EARS-04 rev. 1:** they never overlap. One caveat: the Glossary's definition of "completed" (NC-2).

Line numbers refer to the current working-tree files. "Spine" means `ARCHITECTURE-SPINE.md`.

## 1. Status of HIGH findings

| ID | Source | Status | Evidence (current text) |
| --- | --- | --- | --- |
| H-1 | PRD re-check | **Resolved** | PRD L377 now says §11 "quotes their text **verbatim as last approved**: the full approved PRD is git commit `f0f6934`, plus the one later approved addition quoted in §11.5". The full texts are restored: §11.1 Glossary, §11.2 prior MVP texts, §11.3 deferred FRs, §11.4 metrics, and §11.5 scope notes, including the post-f0f6934 "Full-fidelity PDF preview" note (L685–690). A mechanical check matched every non-blank line of §11.1, §11.2, §11.4, and the f0f6934 part of §11.5 to a line in f0f6934. The only exception is the §11.2 heading itself. See section 2. |
| F1 | reconcile | **Resolved** | Spine AD-9 L154 has `Listing = … 'capped' …`. AD-13 L234: "at 50,000 entries, listing ends as `capped`. It is a state, not an error". AD-11 rule 8 → `noMatchCapped`, and the cap notice applies "when `listing === 'capped'`" (L205). PRD FR-12 L234 gives the wording, and NFR-4 L279 says "Listing stops at 50,000 file entries (capped), and Find says so". `noMatchFinal` can't be reached while capped, because rule 8 comes before rule 10. |
| F2 | reconcile | **Resolved** | `foldersUnreadable` is in the snapshot (AD-9 L160). AD-13 L233: "An unreadable subfolder increments `foldersUnreadable`". AD-11 has the unreadable notice (L206) and rule 9 → `noMatchPartial`. The typed `folderUnreadable` event is in AD-4 L88. PRD FR-12 L235 gives the wording. |
| R1 | rubric | **Resolved** | The Stack pins TypeScript 6.0.3 (L387), "matches the create-vite 9.3.1 `react-ts` starter", with ESLint 10.13.0, typescript-eslint 8.72.0, and eslint-plugin-boundaries 7.3.0 (L391–393). Memlog L46 records the user's decision, citing "typescript-eslint peers TS <6.1". AD-2 L67 adds "a lint fixture that deliberately violates it and must fail". AD-15 L276 adds a fixture for each ban. |
| R2 | rubric | **Resolved** | AD-9 L165–173 defines the `SessionController` interface: commands return `void`, and state is read "only through `getSnapshot()` and `subscribe()`". `seq` is in the snapshot. AD-5 L106–107: the UI depends only on the interface, and the S3 fallback is "a proxy that implements `SessionController` … neither the UI nor the core's rules change". AD-2 L65: no concrete controller class in the UI. |
| A1 | adversarial | **Resolved** | AD-8 L141: "compares its captured generation with the current one **after every `await`**", with writes only through `addEntries(generation, events)`, "which ignores stale generations". L140 cancels "any pending batched-snapshot timer". Nit: closing the old iterator with `return()` isn't stated. Harmless, because stale writes are ignored. |
| A2 | adversarial | **Resolved** | AD-8 L139: "`generation` increments **only** when `FolderPicker.pick()` returns `granted`. A `notOpened` result changes nothing except setting the FR-2 hint". See NC-1 for a new PRD conflict this creates. |
| A3 | adversarial | **Resolved** | AD-9 L152: `type FileType = 'text' \| 'pdf' \| 'image' \| 'other'`. `setTypeFilter(types: ReadonlySet<FileType>)` takes the whole set, and an empty set means no filter (L175). The snapshot carries `typeFilter` (L161). AD-12 L218: the checkboxes are controlled by `snapshot.typeFilter`. |
| A4 | adversarial | **Resolved** | There is one `SessionSnapshot` (AD-9 L157–164) with `primary`, `filtersHidden`, `stillUpdating`, and a flat `generation`. There is no `message` or `error` field: session errors are now `listing`/`primary` states (AD-13). `stillUpdating` = `listing === 'running'` (AD-11 L202), independent of `primary`. |
| A5 | adversarial | **Resolved** | AD-4 L86–94 types `FileSourceEvent` as `entry` (with `modified: number \| 'unavailable'`), `folderUnreadable`, or `accessLost`. "No browser exception reaches the core." The type is derived in the core (Conventions L375). |
| Tech #2 | tech-currency | **Resolved** | Same evidence as R1. TS 6.0.3 is aligned with the starter, and oxlint is replaced by ESLint (Stack L391; memlog L46). |
| Tech #15 | tech-currency | **Resolved** | AD-14 L254–255: "`data:` is never added to the CSP. The build sets `build.assetsInlineLimit: 0`". The L260 build test asserts "`dist/` contains no `data:` URLs". The Stack (L389) repeats `assetsInlineLimit: 0`. |

**Counts:** 12 resolved · 0 partial · 0 open.

### PRD re-check mediums

| ID | Status | Evidence |
| --- | --- | --- |
| M-1 | **Resolved (nit)** | The §0 table (L32–34) now lists FR-2 and FR-3 under "MVP, changed (prior text in §11.2)". Everything still labelled "unchanged" is byte-identical to f0f6934: FR-4, FR-17, NFR-2, NFR-3, NFR-7. §11.2 holds the full prior texts, so the removed parts are recorded: the FR-2 guide link, the FR-3 tree, index, and selection, and NFR-1/6/8. Nit: FR-15 (L240–246) still has no "*Deferred from the original FR-15 (§11)…*" line, unlike FR-1, FR-6, FR-9, FR-12, and FR-21. |
| M-2 | **Resolved** | NFR-3 L276 again lists "no file content, file metadata, …", identical to f0f6934 L406. |
| M-3 | **Resolved** | Option (a) was applied. FR-9 L215: "`projectreport` matches `Project_Report_Q3.docx`, because the name also matches with its separators … removed". Glossary L130 agrees. |
| M-4 | **Resolved** | (a) L232: with an empty query, the notice "counts the file entries the filter excludes". (b) L230: "When the filter hides nothing, no notice is shown." (c) L233: "This folder has no files". (d) L229: "replaces … A file hidden by the filter still matches the name", which replaces "takes precedence". |
| M-5 | **Partial** | The cap (L234), unreadable folders (L235), and lost access (L236) are all specified, and the Glossary lists the end states (L112–116). But the fix asked the Glossary to say that "completed" covers a listing that ended with skipped subfolders. Instead, L114 defines **completed** as "every folder was read". That contradicts FR-12 L235 and AD-11 rule 9. See NC-2. |
| M-6 | **Resolved (nit)** | The §0 EARS-01 row (L40) now names both edge cases: a separators-only query counts as empty, and words match in any order. Nit: the suggested matching note in FR-9 was not added. The Glossary (L129) covers it. |

## 2. §11 against git commit f0f6934

**Verdict: verbatim.**

- **Method:** the `#### FR-n` blocks were extracted from `git show f0f6934:…/prd.md` and from the current §11.3 (L527–662), then compared line by line after trimming trailing whitespace. The whole §11.3 span was also diffed against the concatenated f0f6934 sections §4.2 (L192–203), §4.4–§4.5 head (L247–328), FR-18 (L337–349), and §4.6–§4.7 (L350–401).
- **Results:**

  | FR | Result |
  | --- | --- |
  | FR-5 | identical |
  | FR-7 | identical |
  | FR-10 | identical |
  | FR-11 | identical |
  | FR-13 | identical |
  | FR-14 | identical |
  | FR-16 | identical |
  | FR-18 | identical |
  | FR-19 | identical |
  | FR-20 | identical |
  | FR-22 | identical |

  The full-span diff differs only by one trailing blank line. FR-5's block is also byte-identical (`cmp`). The section descriptions and the "Out of Scope" note on FR-22 are verbatim as well.
- **Also verified:** the §11.2 prior texts of FR-1, 2, 3, 6, 8, 9, 12, 15, and 21 are identical to f0f6934. The NFR-1, 4, 5, 6, and 8 lines, every §11.1 Glossary line, every §11.4 metric line, and the f0f6934 part of §11.5 each appear verbatim in f0f6934.
- **Cosmetic (low):** §11.3 keeps the original headings `### 4.2 Folder Tree and File Details`, `### 4.4 Content Search`, `### 4.5 Filters`, `### 4.6 …`, and `### 4.7 Rescan`. They sit at the same level as `### 11.3`, so the PRD now has two different "4.2", "4.4", and "4.5" headings. This is the price of verbatim quoting. A one-line note under §11.3 ("original section numbers kept") would remove the ambiguity.

## 3. New contradictions: PRD FR-12 and FR-9 against spine AD-9, AD-11, and the matching convention

### NC-1 — Medium — A cancelled picker during an active session: the PRD returns to the start state, but the spine keeps the session
- **PRD FR-2 (L156, L159):** "When the folder picker closes without granting access, Find returns to the start state". The hint "stays until the user chooses a folder successfully".
- **Spine AD-8 (L139):** "A `notOpened` result changes nothing except setting the FR-2 hint." AD-11 maps `start` only to "none yet, or listing `stopped`" (rule 2).
- **Conflict:** with an active session, the user clicks **Choose folder** (FR-3 implies it stays available) and cancels. The spine keeps the current results, with `hint: 'folderNotOpened'` set. The PRD ends the session and shows the start state. AD-11 also has no notice for the hint outside the start state, so where it renders mid-session is undefined. Reconcile F3 asked for this choice to be confirmed with the PM. The spine decided it, but FR-2 wasn't updated.
- **Fix:** add a consequence to FR-2: "If a folder is already selected, a cancelled picker keeps the current session and shows the hint next to it." Or change AD-8 to match the PRD. Either way, say where the hint renders when `primary` isn't `start`.

### NC-2 — Medium — The Glossary's "completed" excludes listings with unreadable folders, but FR-12 and AD-11 treat them as completed
- **PRD Glossary L113:** "**completed:** every folder was read". L116: an unreadable subfolder "is counted … and listing continues".
- **PRD FR-12 L235:** "With no matches after listing completes, Find shows 'No matching files found (N folders couldn't be read)'". **Spine AD-11 rule 9:** "Listing is `completed` and `foldersUnreadable > 0` → `noMatchPartial`".
- **Conflict:** read literally, a listing with one unreadable folder is not *completed*, *capped*, or *stopped*. It has no end state. EARS-03's "listing has completed" then never holds for it, and rule 9 can't be reached. This is the clarification that M-5 asked for, and the opposite text was written.
- **Fix:** Glossary L113: "**completed:** every folder was visited (some may be unreadable folders)".

### NC-3 — Low — "This folder has no files" can be shown when some folders couldn't be read
- **Where:** PRD FR-12 L233; spine AD-11 rule 3 (`emptyFolder`), which comes before rule 9.
- **Problem:** if every subfolder is unreadable, Find states "This folder has no files", with "N folders couldn't be read" as a separate notice. The no-match case qualifies its message with the count (L235), but the empty-folder case doesn't. This is in tension with Principle 1 (L57). Both documents agree, so it is an internal gap rather than a mismatch between them.
- **Fix:** make `emptyFolder` require `foldersUnreadable === 0`. Otherwise fall through to `noMatchPartial`, or add the qualifier "(N folders couldn't be read)".

### NC-4 — Low — The unreadable count appears twice in the `noMatchPartial` case
- **Where:** spine AD-11 L203–206 (notices are "shown alongside `primary`, independent of it", including the unreadable notice); PRD FR-12 L235 (the notice is "alongside the results", and the no-match message itself embeds "(N folders couldn't be read)").
- **Problem:** the spine shows "No matching files found (N folders couldn't be read)" plus the separate notice "N folders couldn't be read". The PRD can be read as showing the notice only when there are results.
- **Fix:** say either that the unreadable notice is suppressed when `primary === 'noMatchPartial'`, or that the `noMatchPartial` text is just "No matching files found" and the notice carries the count.

### NC-5 — Low — An empty query with an active filter falls through to `stillSearching` while nothing has been listed yet
- **Where:** spine AD-11 rules 4–7; PRD FR-12 L227/L232; EARS-04 ("a non-empty search term").
- **Problem:** with an empty query, a filter on, listing `running`, and 0 entries listed so far, no earlier rule matches: `prompt` needs no filter, and both visible results and `filtersHidden` are 0. AD-11 therefore returns `stillSearching`, "No matches yet. Still searching…", for a state that EARS-04 and FR-12 don't cover. Separately, the PRD notice text "Filters hide N matches" counts *file entries* when the query is empty (L232), so the word "matches" doesn't fit there.
- **Fix:** in FR-12, either accept `stillSearching` for this case or define a message for it. Give the empty-query notice its own wording, for example "Filters hide N files".

### NC-6 — Low — Nothing clears the FR-2 hint when a folder is chosen successfully
- **Where:** spine AD-8 L140 (on a new generation, it clears "the entries, the query, the type filter, and the unreadable-folder count"); AD-13 L236 (`accessLost` sets `hint`); PRD FR-2 L160 ("The hint stays until the user chooses a folder successfully").
- **Fix:** add "and resets `hint` to `'none'`" to AD-8 L140.

### FR-9 against the spine's Matching convention — no contradiction
- **PRD:** FR-9 L207, L215–216; Glossary L129–130: "contains every query word … also matches when it does so with its separators removed".
- **Spine:** Conventions L376: "contains every query word, either as written or with its own separators removed", with NFC, `toLocaleLowerCase('en')`, and empty pieces dropped.
- **Two readings:** "the whole condition holds against the separator-free name" (PRD) and "each word matches either form" (spine). Query words never contain separators, so any word found in the raw name is also found in the separator-free name. Both readings therefore reduce to "every query word is in the separator-free name", and they are equivalent.
- **Residual (low, carried over from A15):** neither document fixes what "spaces" means (U+0020 only, or all Unicode whitespace). The spine also doesn't list "its own separators" explicitly. Suggest one exported `splitQuery`/`stripSeparators` pair using one character class.

## 4. EARS-03 rev. 3 and EARS-04 rev. 1 under `capped` and `stopped`

**Result: they never overlap.**

- **Preconditions:**
  - EARS-03 needs "listing has completed".
  - EARS-04 needs "listing is in progress".
  - In the spine, `listing` is a single enum value (AD-9 L154), and the two map to distinct values: `completed` and `running`.
  - `stillSearching` is reached only for `running` (rule 7), and `noMatchFinal`/`noMatchPartial` only after rules 7 and 8 have failed.
- **Under `capped`:** neither EARS item applies. Rule 8 returns `noMatchCapped`, and PRD L234 says "EARS-03's message never appears on its own, because listing did not complete". `running` → `capped` is a one-way transition, so EARS-04 stops applying at the cap.
- **Under `stopped`:** rule 2 returns `start` before any search rule is evaluated, so neither EARS item applies.
- **Caveat 1 (NC-2):** under the literal Glossary, a listing with unreadable folders is never *completed*, so EARS-03 would never apply to it. This doesn't create an overlap, but it does create a gap where neither applies.
- **Caveat 2 (tests):** "No matching files found in the first 50,000 files" (capped) and "No matching files found (N folders couldn't be read)" (partial) both start with the EARS-03 string. EARS tests must assert the exact message text, not a substring, or a capped run could pass the EARS-03 test.
