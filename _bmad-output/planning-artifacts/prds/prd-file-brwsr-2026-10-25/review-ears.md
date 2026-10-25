---
title: "EARS Conformance Review v2: PRD Find"
reviewed: 2026-10-25
supersedes: review-ears-v1.md
inputs:
  - docs/product/earsRequirments (EARS-03 rev. 1, EARS-04 new, revision history)
  - _bmad-output/planning-artifacts/prds/prd-file-brwsr-2026-10-25/prd.md
  - _bmad-output/planning-artifacts/prds/prd-file-brwsr-2026-10-25/review-ears-v1.md
scope: EARS-01 to EARS-04, the EARS file's Project Constraints, and the status of every v1 finding
---

# EARS Conformance Review v2: PRD "Find"

## 1. Verdicts

| EARS | Pattern (as labelled) | Carried by | Acceptance test reproduced | Verdict |
|---|---|---|---|---|
| EARS-01 (name contains search term) | Event-driven, When ... shall | FR-8, FR-9 (substring kind), FR-11 | Yes. Same files, action, and result. "Only" is still written as "and not `photo.jpg`" (F-10, low). | **Satisfied with caveats.** Substring matching now honours "names contain the search term" for every query that yields at least one query word. Queries made only of separators or spaces are undefined (N-02). Lows F-10 and F-13 remain. |
| EARS-02 ("Select a file to preview") | State-driven, While ... shall | FR-20 (last bullet) | Yes. Exact message, folder selected, nothing selected. | **Satisfied with caveats.** Unchanged since v1: "preview" vs "preview panel" (F-06) and the unlogged "folder selected" narrowing (F-07) are still open. |
| EARS-03 rev. 1 ("No matching files found") | Unwanted behaviour, If ... then ... shall | FR-12 (bullets 1 and 3) | Yes. Same fixture, the precondition "after listing and indexing complete", the query, and the exact message. | **Satisfied with caveats.** FR-12 mirrors the revised statement faithfully, including the cancelled branch. But the shared condition "or indexing was cancelled" overlaps EARS-04 while listing is still running (N-01, high). |
| EARS-04 ("No matches yet. Still searching…") | Labelled state-driven; actually While + If | FR-12 (bullet 2), FR-22, UJ-1 | Yes. Same fixture, query, and exact message, including the transition to EARS-03. | **Satisfied with caveats.** Message and precondition match exactly. Caveats: overlap with EARS-03 (N-01), an acceptance test that cannot be reproduced reliably on a two-file fixture (N-03), and form issues (N-05). |

**Project Constraints:** Still all covered. NFR-3 now lists session storage and the origin private file system (v1 F-09 resolved). NFR-1's "after the initial page load" scoping is still undocumented (F-11, low).

**New finding counts:** critical 0, high 1, medium 2, low 3.

**v1 finding status (17 findings):** resolved 6, partially resolved 2, open 9. The v1 out-of-scope note (name the edit-distance metric) is also resolved: FR-10 now names Damerau-Levenshtein (OSA).

## 2. Traceability

### 2.1 Chain

- The EARS file keeps the original EARS-03 statement and its acceptance test verbatim under "Revision History", marked superseded. The history entry gives the reason (a false "no match" during listing or indexing) and points to PRD §0.
- PRD §0 now has an "EARS traceability" table with status, carrying FRs, and the FR that holds each acceptance test. The EARS-03 row says it was revised and that the original is in the EARS file's revision history. **The original EARS-03 remains traceable.**
- Remaining gaps:
  - The table does not restate the statements. That is acceptable now that the EARS file is the single source and the PRD carries no semantic deviations for EARS-01, EARS-03, or EARS-04.
  - EARS-02 is marked "Unchanged", but the PRD still narrows it to "with a folder selected" without logging that (F-07).
  - The EARS file names the project "Smart Local File Browser"; §0 still does not connect that name to "Find" (F-17).

### 2.2 EARS-01 -> FR-8, FR-9

**Acceptance fidelity.** Input `report.pdf`, `photo.jpg`, `report-final.docx`; query `report`; FR-8 expects `report.pdf` and `report-final.docx` and not `photo.jpg`. This is faithful for the fixture. "Only" is still not written as "exactly" (F-10).

**Does substring matching fully honour "names contain the search term"?** Yes, with one gap.
- §3 Query splits the query by the token rules. Every token of the search term is a contiguous piece of that term, because splitting only removes separators or cuts at case changes. So if a name contains the term (ignoring case), it contains every query word, and §3 Name match ("text anywhere in the name") makes it a name match.
- Checked cases: `port` → `report.pdf`; `report` → `annualreport2025.pdf`; `report-final` → `report-final.docx`; `report2025` (letters and digits are not split) → `annualreport2025.pdf`; `ProjectReport` → `ProjectReport.txt`.
- The PRD returns a superset: word order is free, and typo and content matches are added. That is consistent with "all files whose names contain"; it only matters for "Only" on the EARS fixture, checked below.
- **Gap:** a non-empty term with no query words, such as `-`, `...`, or spaces only, is undefined. Read literally, "every query word matches" is then vacuously true for every file (N-02).

**Could any match kind add extra files to the EARS-01 fixture?** No.

| Match kind | Can `report` match `photo.jpg`? |
|---|---|
| exact / partial | No. Its tokens are `photo` and `jpg`; neither equals `report` nor starts with it. |
| substring | No. `photo.jpg` does not contain `report`. |
| typo | No. `report` has 6 letters, so 1 edit is allowed (FR-10). Its distance to `photo`, `jpg`, and every prefix of them is at least 4. |
| content | No. `photo.jpg` is an image, and images are not indexed. `report-final.docx` (type other) is not indexed either. |

Result: exactly `report.pdf` and `report-final.docx`. **The test passes.**

### 2.3 EARS-02 -> FR-20

Unchanged from v1. Input, action, and exact message match. FR-20 still says "the preview shows" (the Glossary defines a preview only for a selected file entry, F-06), and it still adds "with a folder selected" without a traceability note (F-07).

### 2.4 EARS-03 rev. 1 -> FR-12

| Aspect | EARS file | FR-12 | Agree? |
|---|---|---|---|
| Condition | no files match a non-empty term, and the search has finished | `xyz123` shows no file entries | Yes |
| "Finished" (normal) | listing and indexing have completed | after listing and indexing complete | Yes |
| "Finished" (cancel) | or indexing was cancelled | "If indexing was cancelled (FR-13) and nothing matches" | Yes, but both share the overlap in N-01 |
| Message | "No matching files found" | "No matching files found" (plus a second line after a cancel) | Yes. The second line is an addition, not a conflict. |
| Area | search results area | result list | Yes in meaning; "search results area" is still not a Glossary term (F-15) |
| Acceptance fixture and action | `report.pdf`, `photo.jpg`; wait for completion; `xyz123` | same | Yes |

The filter rule in FR-12 says "Filters hide N matches" takes precedence over the EARS-03 and EARS-04 messages. That does not conflict with EARS-03: if filters hide matches, files do match the term, so EARS-03's condition is false. v1 F-04 is resolved.

### 2.5 EARS-04 -> FR-12

| Aspect | EARS file | FR-12 | Agree? |
|---|---|---|---|
| Precondition | While listing or indexing is in progress | while listing or indexing is in progress | Yes, exact |
| Condition | no files match a non-empty term | `xyz123` shows no file entries | Yes |
| Message | "No matches yet. Still searching…" | "No matches yet. Still searching…" | Yes, exact (also UJ-1 and §0) |
| Transition | "Once indexing completes, the message changes to 'No matching files found'" | "It changes to 'No matching files found' once both complete" | Equivalent in practice, worded differently (N-06) |
| Rescan | not stated | FR-22: "EARS-04 applies" during a rescan | Consistent extension |

No leftover copy of the old message ("Still searching file contents…") remains in the PRD.

### 2.6 Project Constraints -> NFR-1 to NFR-3

| EARS constraint | PRD | Status |
|---|---|---|
| All processing local | NFR-1, principle 5 | Covered |
| No network requests or uploads | NFR-1 ("After the initial page load ...") | Covered. The scoping is still undocumented (F-11, low). |
| Read-only access | NFR-2, FR-1 | Covered |
| No persistent storage of contents or metadata | NFR-3 now names local storage, session storage, IndexedDB, Cache Storage, the origin private file system, and cookies | Covered (F-09 resolved). The list is still closed rather than "including"; Cache Storage covers service-worker caches. |
| Explicit folder selection | FR-1 | Covered |

## 3. New Findings

### N-01 [HIGH] EARS-03 rev. 1 and EARS-04 overlap when indexing is cancelled before listing finishes

- **Location:** EARS file, EARS-03 rev. 1 and EARS-04; PRD FR-12 bullets 2 and 3; FR-13; FR-22.
- **Quote:** EARS-03: "the search has finished (listing and indexing have completed, **or indexing was cancelled**)". EARS-04: "While **listing** or indexing is in progress". FR-12: "If indexing was cancelled (FR-13) and nothing matches, Find shows 'No matching files found'".
- **Problem:** Listing and indexing both start when access is granted (FR-1, FR-13), so the user can cancel indexing while listing is still running. This is likely for large folders of up to 50,000 file entries.
  - In that state both conditions are true. EARS-03 requires "No matching files found", and EARS-04 requires "No matches yet. Still searching…" in the same area. Two requirements conflict, and no implementation can pass both.
  - Choosing the EARS-03 message re-creates the defect the revision was meant to remove: a false "no match" while names are still being listed (principle 1, "When a search hasn't finished, Find never claims 'no match'").
  - "Indexing was cancelled" is also sticky. During a rescan after an earlier cancel (FR-22), it is unclear whether the cancel still counts while the new listing and indexing run.
- **Fix:** In the EARS file and FR-12, make listing completion mandatory: "the search has finished (listing has completed, and indexing has completed or was cancelled)". In FR-12 bullet 3, write "If listing has completed, indexing was cancelled, and nothing matches...". In FR-22, state that a rescan clears the cancelled state.

### N-02 [MEDIUM] A non-empty query with no query words is undefined, and literally matches every file

- **Location:** §3 Query, Token, Name match; FR-12 (empty-query bullets); EARS-01 and EARS-03 "non-empty search term".
- **Quote:** "It is split into query words by the token rules"; "A file entry where *every* query word matches its name".
- **Problem:** Spaces, underscores, hyphens, and dots are separators. So a term such as `-`, `_`, `...`, or spaces only is non-empty to EARS but produces zero query words.
  - Read literally, "every query word matches" is vacuously true, so every file entry becomes a name match. That breaks EARS-01 (`-` should return only names containing `-`) and EARS-03 (`...` with no matching name should show "No matching files found").
  - Alternatively, the query is treated as empty, and FR-12 then shows the folder tree. That is a reasonable choice, but it is an unlogged deviation from "non-empty".
  - v1 F-15 raised whitespace-only queries. The new token-based query splitting widens the gap to separator-only queries.
- **Fix:** Add to §3: "A query that yields no query words (empty, whitespace only, or separators only) is an **empty query**." Record this reading of "non-empty" in the §0 traceability table for EARS-01, EARS-03, and EARS-04.

### N-03 [MEDIUM] The EARS-04 acceptance test cannot be reproduced reliably as written

- **Location:** EARS file, EARS-04 acceptance test; FR-12 bullet 2.
- **Quote:** "Input: A folder containing `report.pdf` and `photo.jpg`, with indexing still in progress. Action: Search for `xyz123` before indexing completes."
- **Problem:** Indexing one small PDF takes milliseconds on the reference machine (NFR-4), so a tester cannot reliably type `xyz123` before it finishes. The test is timing-dependent, as the original EARS-03 test was. Neither document says how to hold Find in the in-progress state.
- **Fix:** Name a deterministic method in FR-12 and the EARS test: either a test hook that pauses the indexer (a unit or integration seam, consistent with NFR-8), or a fixture with `report.pdf` large enough that indexing visibly runs. Also state the observable that ends the in-progress state: the FR-15 *still updating* indicator disappears.

### N-04 [LOW] No match kind for a query word that spans a token boundary

- **Location:** §3 Match kind; FR-11 bullet 1.
- **Quote:** "**substring:** every query word appears somewhere in the name, and at least one appears only inside a token".
- **Problem:** `tre` matches `ProjectReport.txt` as a name match ("text anywhere in the name", case ignored) because it spans `Project|Report`. It is not exact or partial, and it does not appear "inside a token", so no match kind applies. FR-11 requires every result to be labelled. EARS-01 itself is still met, because the file is returned.
- **Fix:** Define substring as "every query word appears in the name, and at least one appears neither as a token nor as a token beginning".

### N-05 [LOW] EARS form quality of the revised statements

- **Location:** EARS file, EARS-03 rev. 1 and EARS-04.
- **Problem:**
  - **EARS-03 rev. 1** puts a state ("the search has finished") inside the If clause of an unwanted-behaviour pattern and defines it in a parenthetical. The cleaner complex form is "While the search has finished (...), if no files match a non-empty search term, then the system shall ...". A no-match result is still a normal outcome rather than an unwanted event (v1 F-15).
  - **EARS-04** is labelled "State-driven", but it is a complex requirement (While + If), and its If clause has no "then". The statement says nothing about the transition to EARS-03, which only its acceptance test asserts.
  - Both statements are unambiguous about message and area, and both use "non-empty", which N-02 shows is underdefined.
- **Fix:** Relabel EARS-04 "Complex (state-driven + unwanted behaviour)" and write "..., if ..., then the system shall ...". Restate EARS-03 rev. 1 in the While/If form. Optionally add to EARS-04: "When listing and indexing complete with still no match, EARS-03 applies."

### N-06 [LOW] The EARS-04 transition wording differs slightly between the files

- **Location:** EARS file, EARS-04 expected result; FR-12 bullet 2.
- **Quote:** EARS: "Once indexing completes, the message changes". FR-12: "once both complete".
- **Problem:** The two are equivalent in practice, because indexing cannot finish before listing has reached every file. FR-12 is the more precise wording, but the texts differ.
- **Fix:** Align the EARS file to "Once listing and indexing complete".

## 4. Status of v1 Findings

| v1 | Sev. | Status | Evidence |
|---|---|---|---|
| F-01 Token-beginning matching contradicts "contain" | HIGH | **Resolved** | §3 Name match adds "text anywhere in the name". The **substring** match kind was added and ranked (§3 Rank, FR-11). FR-9 test: `report` matches `annualreport2025.pdf`, and `port` matches `report.pdf`. |
| F-02 Query splitting undefined | HIGH | **Resolved** | §3 Query: split "by the token rules". FR-9 test: `report-final` matches `report-final.docx`. The residual separator-only case is new finding N-02. |
| F-03 FR-12 changed EARS-03's message | HIGH | **Resolved** (v1 option b) | EARS-03 revised, EARS-04 added, original kept, §0 table records both. FR-12 now matches the revised EARS text exactly. A residual overlap in the revised condition is new finding N-01. |
| F-04 Filter message precedence | MEDIUM | **Resolved** | FR-12: "This message takes precedence over the EARS-03 and EARS-04 messages". "When active filters hide matches" implies N ≥ 1. |
| F-05 Message after a cancelled index | MEDIUM | **Resolved** | FR-12 bullet 3 and EARS-03 rev. 1 cover the cancel. The listing-still-running gap is N-01. |
| F-06 "Preview panel" vs Glossary "preview" | MEDIUM | **Open** | §3 Preview and the FR-20 wording are unchanged. |
| F-07 EARS-02 narrowing not logged | MEDIUM | **Open** | The §0 table marks EARS-02 "Unchanged". FR-20 still adds "with a folder selected". |
| F-08 No restated statements or deviation log | MEDIUM | **Partially resolved** | The §0 traceability table was added with status, carriers, and acceptance FRs. There is still no deviation note for EARS-02 (F-07) or for the reading of "non-empty" (N-02). |
| F-09 NFR-3 storage list | MEDIUM | **Resolved** | Session storage and the origin private file system were added. |
| F-10 "Only" vs "and not" | LOW | Open | FR-8 is unchanged. |
| F-11 NFR-1 scoping undocumented | LOW | Open | NFR-1 is unchanged. |
| F-12 Fixture content unspecified | LOW | Open | The content of `report.pdf` is still unspecified. It matters for the EARS-03 and EARS-04 tests. |
| F-13 "All files" vs the 50,000 cap | LOW | Open | NFR-4 is unchanged. |
| F-14 EARS-01 form ("enters", all vs only) | LOW | Partially resolved | The "contain" semantics are now honoured. "Enters" and the all-vs-only mismatch remain in the EARS file. |
| F-15 EARS-03 pattern and ambiguity | LOW | Open | Still If/then for a normal outcome. Blank queries are undefined (now widened, N-02). "Search results area" and "search box" are not Glossary terms. |
| F-16 Deselection rules | LOW | Open | Not addressed. FR-22 ("the preview shows 'This file is no longer in the folder'") adds a case where it is unclear whether a file entry is still selected, and so whether EARS-02 applies. |
| F-17 Project name mismatch | LOW | Open | §0 does not mention "Smart Local File Browser". |

**Totals:** resolved 6 (3 high, 3 medium), partially resolved 2 (1 medium, 1 low), open 9 (2 medium, 7 low). All v1 high findings are resolved.

## 5. Observation (not counted)

Substring matching has no minimum length, so a single-letter query such as `r` returns almost every file, ranked as substring matches. This is consistent with EARS-01 read literally, but it bears on SM-C1 (wrong files in the top 5). Tune it alongside OQ-4 if needed.
