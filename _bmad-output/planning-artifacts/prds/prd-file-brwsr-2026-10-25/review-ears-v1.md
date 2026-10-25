---
title: "EARS Conformance Review: PRD Find"
reviewed: 2026-10-25
inputs:
  - docs/product/earsRequirments
  - _bmad-output/planning-artifacts/prds/prd-file-brwsr-2026-10-25/prd.md
scope: EARS-01, EARS-02, EARS-03 and the EARS file's Project Constraints
---

# EARS Conformance Review: PRD "Find"

## 1. Verdicts

| EARS | Pattern | Carried by | Acceptance test reproduced | Verdict |
|---|---|---|---|---|
| EARS-01 (event-driven, name search) | When ... shall | FR-8 (with FR-9, FR-10, FR-11 shaping semantics) | Yes: same files, same action, same result. "Only" softened to "and not `photo.jpg`" (equivalent for this fixture). | **Satisfied with caveats.** The acceptance test passes. The statement's "all files whose names contain the search term" is contradicted by token-beginning matching (F-01) and by the undefined query splitting rule (F-02). |
| EARS-02 (state-driven, empty preview) | While ... shall | FR-20 (last bullet) | Yes: folder selected, nothing selected, exact message "Select a file to preview". | **Satisfied with caveats.** The term "preview panel" is not reconciled with the Glossary's "preview" (F-06). The PRD adds a folder-selected precondition that the EARS statement lacks (F-07). |
| EARS-03 (unwanted behavior, no match) | If ... then ... shall | FR-12 (first two bullets) | Partly. Same fixture, query, and exact message, but the PRD adds the precondition "after indexing completes". | **Satisfied with caveats.** While indexing runs, the PRD shows a different message for the same no-match condition (F-03). Message precedence with filters and after a cancelled index is undefined (F-04, F-05). |

**Project Constraints:** All five are covered: local only (NFR-1, §1 principle 5), no network (NFR-1), read-only (NFR-2, FR-1), no persistence (NFR-3, FR-3), and explicit folder selection (FR-1). There are two small gaps in wording (F-11, F-12).

**Finding counts:** critical 0, high 3, medium 6, low 8.

## 2. Traceability Detail

### EARS-01 -> FR-8

- EARS input: `report.pdf`, `photo.jpg`, `report-final.docx`. PRD FR-8 uses the same files.
- EARS action: "Enter `report` in the search field." PRD: "the query `report`". FR-8 adds "updates as the user types, without pressing Enter", which matches "When the user enters a search term".
- EARS expected: "**Only** `report.pdf` and `report-final.docx` appear". PRD: "returns `report.pdf` and `report-final.docx`, and not `photo.jpg`". These are equivalent for a three-file fixture, but the wording is weaker in general (F-10).
- Over-matching check with the EARS-01 input under PRD rules:
  - Name tokens are `report`/`pdf`, `photo`/`jpg`, and `report`/`final`/`docx`.
  - `report` (6 letters, 1 edit allowed by FR-10) is not within 1 edit of `photo`, `jpg`, `pdf`, `final`, or `docx`. So the typo rule adds no extra result.
  - Content search adds nothing: `photo.jpg` is an image and `report-final.docx` is type **other**, so neither is indexed.
  - **Result: no extra files. The test passes.**
- Under-matching check (the statement's "all ... contain"): the PRD misses some names that contain the term. See F-01 and F-02.

### EARS-02 -> FR-20

- Input, action, and exact message all match: "with a folder selected and no selected file entry, the preview shows 'Select a file to preview'".
- The PRD says "the preview" where EARS says "the preview panel". The Glossary defines "preview" as the view *of the selected file entry*, so the word does not apply when nothing is selected (F-06).

### EARS-03 -> FR-12

- Input, query `xyz123`, the expected empty list, and the exact message "No matching files found" all match.
- The PRD adds "after indexing completes" and a different message while indexing runs. The EARS fixture contains `report.pdf`, which the PRD indexes, so the EARS test as originally written can observe "No matches yet. Still searching file contents…" if it runs before indexing finishes (F-03).

### Project Constraints

| EARS constraint | PRD coverage | Note |
|---|---|---|
| All file processing local in the browser | NFR-1, §1 Vision, principle 5 | Covered. |
| No network requests or server uploads | NFR-1, NFR-5, §6 | The PRD scopes this to "after the initial page load" (F-11). |
| Read-only access to user-selected folders | NFR-2, FR-1, §6 | Covered, and stricter than EARS. |
| No persistent storage of file contents or metadata | NFR-3, FR-3, §6 | The storage list is not exhaustive (F-12). OQ-5 (remember only the folder permission) would break NFR-3, but arguably not the EARS constraint, which names only contents and metadata. |
| User must explicitly select the folder | FR-1 ("requires an explicit user action ... never opens the folder picker on its own") | Covered. OQ-1 drag-and-drop would still be an explicit action. |

## 3. Findings

### F-01 [HIGH] Token-beginning matching contradicts EARS-01 "names contain the search term"

- **Location:** §3 Glossary "Name token" / "Name match"; FR-9; FR-8.
- **Quote:** EARS-01: "display all files ... whose names contain the search term". PRD FR-9: "A query word matches a token exactly, as the token's beginning, or within typo tolerance".
- **Problem:** EARS-01 describes substring matching. The PRD matches only at token boundaries. Names that contain the term in the middle of a token are never returned:
  - `annualreport2025.pdf` has the tokens `annualreport`, `2025`, `pdf`. `report` is not a token beginning, and its edit distance to `annualreport` is 6. **It is not returned, although its name contains `report`.**
  - The query `port` does not return `report.pdf`.

  This silently narrows a "shall display all" requirement and goes against Product Principle 1 ("Never hide the right file"). Lowercase concatenated names are common in downloads.
- **Fix:** Choose one option and record it.
  - (a) Add a substring fallback: a query word of 3 or more characters that occurs anywhere inside a name token gives a name match, ranked after token-beginning matches (label **partial**, or a new kind). Add the test "`report` returns `annualreport2025.pdf`".
  - (b) Keep token semantics as a deliberate refinement. Amend EARS-01 to "...whose name tokens begin with, or approximately match, each word of the search term", and record the deviation in a traceability table (see F-09).

### F-02 [HIGH] The query splitting rule is undefined, so `report-final` may not find `report-final.docx`

- **Location:** §3 Glossary "Query"; FR-9.
- **Quote:** "The text the user types into search. It is split into **query words**."
- **Problem:** The Glossary gives detailed splitting rules for *names* (separators, case changes, letter/digit boundaries) but none for the *query*. FR-9's examples use only spaces.
  - If the query is split only at spaces, the query `report-final` is one word.
  - That word matches no single token of `report-final.docx` (`report`, `final`, `docx`). Its edit distance is beyond the 2-edit limit.
  - So a file whose name literally contains the search term is not returned. This breaks EARS-01 on a name taken from the EARS fixture itself. The same problem applies to `report.pdf`, `Q3_Report`, and similar queries.
- **Fix:** Add to the Glossary: "The query is split into query words by the same rules as name tokens." Add FR-9 tests: "`report-final` matches `report-final.docx`" and "`report.pdf` matches `report.pdf`".

### F-03 [HIGH] FR-12 changes EARS-03's message while indexing runs and adds a precondition to its acceptance test

- **Location:** FR-12, bullets 1-2.
- **Quote:** "after indexing completes, the query `xyz123` shows ... 'No matching files found'" and "While indexing is running and there are no matches yet, Find shows 'No matches yet. Still searching file contents…' instead of 'No matching files found'."
- **Problem:** EARS-03 is unconditional: if no files match a non-empty term, the system shall show "No matching files found".
  - The PRD replaces that message with a different one during indexing, and adds "after indexing completes" to the acceptance test. The EARS test has no such precondition.
  - The EARS fixture contains `report.pdf`, which the PRD indexes. The original EARS test is therefore timing-dependent: run immediately after folder selection, it fails on the exact message text.
  - The change is reasonable for the product, but it is an undocumented modification of a source requirement.
- **Fix:** Choose one option.
  - (a) Always show "No matching files found" when the result list is empty for a non-empty query. While indexing runs, add the FR-15 *still updating* indicator as a secondary line ("Still searching file contents…"). This keeps EARS-03 literally true, and the FR-12 test can drop its precondition.
  - (b) Keep the PRD behavior, amend EARS-03 to "If indexing has completed and no file entries match ...", and add a second EARS statement for the indexing state ("While indexing is running, if no file entries match..., the system shall display 'No matches yet. Still searching file contents…'"). Record both in the traceability table.

### F-04 [MEDIUM] Message precedence is undefined when filters hide every match

- **Location:** FR-12, bullet 3.
- **Quote:** "When active filters hide matches, Find shows how many, for example 'Filters hide 12 matches' ... This applies whether the visible result list is empty or not."
- **Problem:** Two cases are unspecified:
  - The query matches files, but filters hide all of them. EARS-03's condition ("no files match a non-empty search term") is false, so "No matching files found" would be misleading. The PRD does not say whether it is suppressed.
  - The query matches nothing and filters are active. It is unclear whether "Filters hide 0 matches" appears.
- **Fix:** Add a precedence rule. "No matching files found" appears only when the query matches zero file entries before filters are applied. When matches exist but all are filtered out, show only "Filters hide N matches" with **Clear filters**. Never show "Filters hide 0 matches".

### F-05 [MEDIUM] The empty-result message after indexing is cancelled is undefined

- **Location:** FR-12, bullets 1-2; FR-13, bullet 3.
- **Quote:** "the result list states that content search is incomplete" (FR-13); "after indexing completes" (FR-12).
- **Problem:** After a cancel, indexing never completes. Neither FR-12 message condition applies: indexing is not "complete", and it is not "running". The EARS-03 behavior is therefore unspecified for this state.
- **Fix:** Add to FR-12: "After indexing is cancelled, an empty result list for a non-empty query shows 'No matching files found' together with the content-search-incomplete notice."

### F-06 [MEDIUM] EARS-02's "preview panel" is not reconciled with the Glossary term "preview"

- **Location:** §3 Glossary "Preview"; FR-20, last bullet.
- **Quote:** Glossary: "**Preview:** The read-only view, inside Find, of the selected file entry." FR-20: "with ... no selected file entry, the preview shows 'Select a file to preview'".
- **Problem:** By the Glossary definition, a preview exists only when there is a selected file entry, so "the preview shows ..." with no selected file entry is self-contradictory. EARS-02 uses "preview panel", a UI region. The Glossary forbids synonyms but defines no region term, so UX has no defined name for the place where the EARS-02 message goes. The same gap exists for EARS-03's "search results area" (F-15).
- **Fix:** Add a Glossary entry: "**Preview panel:** the region of Find that shows the preview of the selected file entry, or the 'Select a file to preview' message when there is none." Reword the FR-20 bullet to "the preview panel shows ...".

### F-07 [MEDIUM] EARS-02's state is underspecified, and the PRD's narrower condition is not stated as a deviation

- **Location:** FR-20, last bullet; §3 "Start state".
- **Quote:** EARS-02: "While no file is selected, the system shall display a 'Select a file to preview' message". Start state: "with no folder tree, result list, or preview".
- **Problem:** The EARS statement applies in every state where no file is selected, including the start state. The PRD's start state shows no preview, so EARS-02 read literally is violated there. The PRD's condition ("with a folder selected") is the sensible one, but it is a silent narrowing.
- **Fix:** Restate EARS-02 in the PRD as "While a selected folder is open and there is no selected file entry, Find shall display 'Select a file to preview' in the preview panel." Note the narrowing in the traceability table.

### F-08 [MEDIUM] The PRD does not restate the EARS statements or log its deviations

- **Location:** §0 Document Purpose.
- **Quote:** "EARS-01 to EARS-03 appear here as acceptance tests."
- **Problem:** Only the acceptance tests are reproduced. The EARS *statements* are absent, and the PRD changes their semantics without saying so (F-01, F-03, F-07). A downstream architect or story writer cannot see that the PRD intentionally departs from the source requirement.
- **Fix:** Add a short "EARS Traceability" subsection, in §0 or as a §11, with one row per EARS requirement:
  - the original statement;
  - the PRD's refined EARS statement, in EARS form (for example: "When the user changes the query, Find shall display every file entry in the selected folder and its subfolders that is a name match or content match, in rank order.");
  - the carrying FR;
  - deviations and their rationale.

### F-09 [MEDIUM] NFR-3's storage list is not exhaustive

- **Location:** NFR-3.
- **Quote:** "to any browser storage (local storage, IndexedDB, Cache Storage, or cookies)"
- **Problem:** The list omits sessionStorage and the Origin Private File System (OPFS).
  - sessionStorage survives a page reload in the same tab, which would undermine FR-3 ("After a page reload, Find shows the start state").
  - OPFS is a likely place for an implementer to spill a content index of up to 500 MB.
  - The EARS constraint "No persistent storage of file contents or metadata" therefore has a loophole.
- **Fix:** Reword to "any browser storage, including but not limited to local storage, session storage, IndexedDB, Cache Storage, the Origin Private File System, service-worker caches, and cookies". Extend the *Verify* step to match.

### F-10 [LOW] The EARS-01 acceptance wording is weaker than "Only"

- **Location:** FR-8, bullet 1.
- **Quote:** "returns `report.pdf` and `report-final.docx`, and not `photo.jpg`"
- **Problem:** This is equivalent for a three-file fixture, but it does not carry EARS's "Only" if the fixture grows.
- **Fix:** Write "returns exactly `report.pdf` and `report-final.docx`".

### F-11 [LOW] NFR-1 scopes "no network" to after the initial load

- **Location:** NFR-1.
- **Quote:** "After the initial page load, Find makes zero network requests"
- **Problem:** The EARS constraint says simply "No network requests or server uploads". Loading the app is an unavoidable request, so the PRD's reading is correct, but it is undocumented.
- **Fix:** Add one line: "The EARS constraint is read as: no requests other than loading Find's own static assets, and never any request that carries file data."

### F-12 [LOW] The fixture file content is unspecified

- **Location:** FR-8 and FR-12 EARS acceptance bullets.
- **Quote:** "in a folder containing `report.pdf` and `photo.jpg`"
- **Problem:** `report.pdf` is content-indexed. If the fixture PDF happened to contain `xyz123`, EARS-03 would fail with a content match. Test results depend on fixture bytes that nobody has specified.
- **Fix:** Specify the fixtures, for example: "`report.pdf` is a text-layer PDF whose text does not contain `xyz123`; `photo.jpg` is any image."

### F-13 [LOW] EARS-01 "all files" vs the NFR-4 capacity cap

- **Location:** NFR-4, bullet 2.
- **Quote:** "Folders larger than that are not supported in the MVP"
- **Problem:** EARS-01 "all files" has no capacity bound. Above 50,000 file entries the PRD does not satisfy it.
- **Fix:** Note the bound in the traceability table. Specify what "not supported" means: whether Find refuses the folder, or searches the first 50,000 entries with a notice.

### F-14 [LOW] EARS form quality: EARS-01 trigger and the statement/test mismatch

- **Location:** docs/product/earsRequirments, EARS-01.
- **Quote:** "When the user enters a search term, the system shall display all files ... whose names contain the search term." Test: "**Only** `report.pdf` and `report-final.docx` appear".
- **Problem:** The When/shall form is well-formed, but:
  - "enters" is ambiguous: each keystroke, or on submit?
  - the statement requires "all" (completeness) while the test asserts "only" (exclusivity), which the statement never requires;
  - "contain" fixes substring semantics, which the PRD does not use.
- **Fix:** In the PRD's restatement (F-08), write "When the user changes the query, ..." and state both completeness and the allowed extras (typo and content matches).

### F-15 [LOW] EARS form quality: EARS-03 pattern choice and ambiguity

- **Location:** docs/product/earsRequirments, EARS-03; FR-12.
- **Quote:** "If no files match a non-empty search term, then the system shall display a 'No matching files found' message in the search results area."
- **Problem:** The If/then form is well-formed, but:
  - A no-match result is a normal outcome, not an unwanted event (failure or invalid input). An event-driven form ("When a search completes with no matching file entries, ...") is a better fit.
  - "match" does not say name, content, or both, or whether filters count.
  - "non-empty" does not settle whitespace-only queries, which the PRD also leaves unspecified.
  - "search results area" and EARS-01's "search field" are not Glossary terms. FR-13 also uses an undefined "search box".
- **Fix:** Restate in the PRD as "When the query is non-blank and no file entry is a name match or content match, Find shall display 'No matching files found' in the result list." Define a blank query (empty or whitespace only) as an empty query. Add "search box" and "result list" (as a region) to the Glossary.

### F-16 [LOW] Deselection rules affecting EARS-02 are unspecified

- **Location:** FR-15, FR-12, FR-20.
- **Quote:** "without changing ... the selected file entry" (FR-15)
- **Problem:** It is unclear whether the preview panel returns to "Select a file to preview" when the selected file entry is filtered out of the result list or the query changes, or when the user selects a folder (not a file) in the folder tree.
- **Fix:** Add an FR-20 consequence: "The selected file entry stays selected until the user selects another file entry or the session ends. Selecting a folder in the folder tree does not clear it."

### F-17 [LOW] Project name: "Smart Local File Browser" vs "Find"

- **Location:** §0; EARS file header.
- **Quote:** EARS: "**Project:** Smart Local File Browser". PRD: "# PRD: Find".
- **Problem:** Traceability survives, because §0 cites the EARS file by path. A reader matching documents by project name would not connect them. This is low impact, and OQ-7 may rename the product again.
- **Fix:** Add one clause to §0: "(the EARS file calls this project 'Smart Local File Browser')".

## 4. Out-of-Scope Observation (not counted)

FR-10 says `reprot` matches `report` with 1 edit. That holds only if a transposition counts as one edit (Damerau-Levenshtein); under plain Levenshtein the distance is 2. FR-10 should name the distance metric. This does not affect the EARS fixtures, but it affects UJ-1 and FR-11's `project reprot` example.
