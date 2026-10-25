# Validation Report — PRD: Find (second run)

- **PRD:** `_bmad-output/planning-artifacts/prds/prd-file-brwsr-2026-10-25/prd.md` (+ `addendum.md`)
- **Rubric:** `_bmad-output/planning-artifacts/prds/prd-file-brwsr-2026-10-25/review-rubric.md`
- **Run at:** 2026-10-25T17:06:19+02:00
- **Grade:** Excellent
- **Previous report:** `validation-report-v1.md`

## Overall verdict

This is a strong, build-ready PRD, and better than the version reviewed in `review-rubric-v1.md`. The edits closed every high finding in the rubric and EARS reviews. Typo distance is now defined (Damerau-Levenshtein, OSA). Query words are split by the same rules as names. A **substring** match kind honors EARS-01's "contain". EARS-03 rev. 1 and EARS-04 are traced in §0 and carried by FR-12. The deliberate DOCX trade-off now reads the same way in the Vision, §2.2, UJ-1, FR-16, §7.2, and SM-1, so it counts as an accepted risk rather than a defect. The new match kinds, rank order, and FR examples agree with one another; every worked example checks out. What remains at risk is narrower. The **substring** tier has no minimum word length and ranks above **content**, so short queries can still bury the content matches the thesis depends on. FR-22 doesn't say what happens to the existing result list during a full rebuild. FR-21's open-in-tab can't keep NFR-5's promise for PDFs, and the UJ-1 hand-off relies on pasting a relative location into File Explorer, which is unverified. None of this blocks UX, but FR-10, FR-21, and FR-22 should be tightened before stories are written.

The EARS re-check (`review-ears.md`) has one high finding, N-01: when indexing is cancelled while listing is still running, EARS-03 rev. 1 ("No matching files found") and EARS-04 ("No matches yet. Still searching…") both apply, so no implementation can pass both. The rubric rates the same issue as low (N-7). A one-line fix has been proposed to the user and is pending approval: make listing completion mandatory in the "finished" condition ("listing has completed, and indexing has completed or was cancelled") in the EARS file and FR-12. The grade follows the rubric, as validate.md specifies; until that fix lands, this is the one open requirements conflict.

The adversarial review (`review-adversarial-general.md`) covered the previous PRD version and is not a current reviewer. Its critical and high findings were tracked in the rubric's *Prior findings status* table: 10 resolved, 2 partially resolved, 2 accepted-risk, 0 open.

## Dimension verdicts
- Decision-readiness — strong
- Substance over theater — strong
- Strategic coherence — strong
- Done-ness clarity — adequate
- Scope honesty — strong
- Downstream usability — strong
- Shape fit — strong

## Severity counts by reviewer
- Rubric (`review-rubric.md`): critical 0, high 0, medium 6, low 7 (+1 uncounted carry-over)
- EARS re-check (`review-ears.md`): critical 0, high 1, medium 2, low 3

## Findings by severity

### Critical (0)
None.

### High (1)
**[EARS re-check]** — N-01. EARS-03 rev. 1 and EARS-04 overlap when indexing is cancelled before listing finishes (EARS-03 rev. 1, EARS-04; FR-12 bullets 2–3; FR-13; FR-22)
Listing and indexing both start on access, so the user can cancel indexing while listing runs. Both conditions are then true and require different messages; no implementation can pass both. Choosing EARS-03 re-creates the false "no match" the revision removed. "Indexing was cancelled" is also sticky across a rescan. Same issue as rubric N-7 (rated low there). Fix proposed to the user; pending approval.
Fix: In the EARS file and FR-12: "the search has finished (listing has completed, and indexing has completed or was cancelled)". In FR-22, state that a rescan clears the cancelled state.

### Medium (8)
**[Strategic coherence]** — N-1. The substring tier can still bury content matches (§3 Match kind, Rank; FR-9; FR-11; SM-3; SM-C1)
Substring has no minimum word length and ranks above content; `tax` matches `syntax.md` and `taxi_receipt.pdf` above the file containing "tax". SM-3 fixtures never meet this competition. Residual of adversarial H-5.
Fix: Give substring a floor (query words of 3 or fewer characters match only as a token or token beginning). Add competing-name fixtures to SM-3 and extend SM-C1.

**[Strategic coherence]** — N-2. SM-8 is measured inside SM-1, but the two protocols conflict (§8 SM-1, SM-8; FR-1)
MyFiles setup either inflates SM-1's timed tasks or SM-8 tests nothing real; `C:\MyFiles` is typically outside Windows Search's default index.
Fix: Run SM-8 as a separate untimed step before SM-1; state whether the seeded folder is MyFiles and, if so, that it is OS-indexed.

**[Done-ness clarity]** — N-3. FR-22 doesn't say what the user sees during a rebuild (FR-22; FR-15; FR-12; NFR-4)
Unclear whether old results stay or are cleared; clearing loses the user's file for up to 60 s, keeping both risks the 200 MB limit.
Fix: State one behavior (e.g. entries stay until listing confirms or removes them; EARS-04 only if the list is empty), or say rescan clears and rebuilds.

**[Done-ness clarity]** — N-4. Typo tolerance against token beginnings has no defined prefix length (FR-10; addendum)
"Similar length" lives only in the addendum. `budgte` vs `budgeting.xlsx` and `from` vs `format` are undecidable from the PRD.
Fix: Compare against the token prefix of the same length as the query word; add a positive and a negative example.

**[Done-ness clarity]** — N-5. FR-21 can't keep NFR-5's and NFR-1's promise for a PDF opened in a tab (FR-21; NFR-5; NFR-1; SM-6)
The browser's PDF viewer is outside Find's control (clickable links, possible scripts). Same gap in-app if pdf.js scripting and links aren't disabled (adversarial M-5).
Fix: Scope NFR-5 to in-app previews and exclude user-initiated link clicks from NFR-1 and SM-6, or open PDFs in a Find-controlled viewer tab.

**[Done-ness clarity]** — N-6. The Copy location hand-off is unverified (FR-21; UJ-1; §3 Location; SM-1)
The copied location is relative; pasting it into Explorer or Finder likely fails, especially on macOS. SM-1 counts this as a valid stop. Residual of adversarial C-2.
Fix: Add an OQ-1 spike item to verify on Windows and macOS; reword UJ-1 and the FR-21 hint; consider copying the file name too.

**[EARS re-check]** — N-02. A non-empty query with no query words is undefined, and literally matches every file (§3 Query, Token, Name match; FR-12; EARS-01, EARS-03)
`-`, `...`, or spaces only yield zero query words, so "every query word matches" is vacuously true.
Fix: Define such a query as an **empty query** in §3 and record this reading of "non-empty" in §0.

**[EARS re-check]** — N-03. The EARS-04 acceptance test cannot be reproduced reliably as written (EARS-04 test; FR-12 bullet 2)
Indexing a small PDF takes milliseconds; the test is timing-dependent.
Fix: Name a deterministic method (an indexer pause hook or a large fixture) and the observable end state (the FR-15 *still updating* indicator disappears).

### Low (10)
**[Done-ness clarity]** — N-7. EARS-03 rev. 1 and EARS-04 overlap after a cancel during listing (FR-12; FR-13; EARS file)
Same issue as EARS N-01 (rated high there).
Fix: "or listing has completed and indexing was cancelled".

**[Done-ness clarity]** — N-8. Whether the extension takes part in name matching is unstated (§3 Token; FR-9)
`pdf` would make every PDF an exact match.
Fix: State whether the extension is a matchable token.

**[Done-ness clarity]** — N-9. A concatenated query doesn't match a separated name (§3 Query, Token; FR-9)
`annualreport` vs `Annual Report 2025.pdf` matches no kind.
Fix: Also compare against the name with separators removed, or list the case in §7.2.

**[Done-ness clarity]** — N-10. FR-22 has two smaller gaps (FR-22)
"Listed at HH:MM" start vs end and format; no message for a file deleted between rescans.
Fix: "time listing completed, local time"; extend the removed-file message to any failed preview or open.

**[Done-ness clarity]** — N-11. The index limit's unit doesn't match its measure (NFR-4)
Characters vs bytes differ by a factor of 2.
Fix: "100 million characters (about 200 MB as UTF-16)".

**[Scope honesty]** — N-12. The Vision still overclaims safety (§1 ¶2 vs §7.2, OQ-9)
"Safe to use on any work computer" vs synced-folder downloads.
Fix: "safe to use on any work computer: it reads only the folder you choose, and changes nothing".

**[Downstream usability]** — N-13. The content match kind contradicts its own tie-break rule (§3 Match kind)
A typo name match that is also a content match should become content, which the definition forbids.
Fix: Define content as "a content match whose name match kind, if any, ranks below content".

**[EARS re-check]** — N-04. No match kind for a query word that spans a token boundary (§3 Match kind; FR-11)
`tre` in `ProjectReport.txt` gets no label.
Fix: Substring = "every query word appears in the name, and at least one appears neither as a token nor as a token beginning".

**[EARS re-check]** — N-05. EARS form quality of the revised statements (EARS-03 rev. 1, EARS-04)
A state inside an If clause; EARS-04 is mislabelled and lacks "then".
Fix: Relabel EARS-04 "Complex" with If/then; restate EARS-03 rev. 1 in While/If form.

**[EARS re-check]** — N-06. The EARS-04 transition wording differs slightly between the files (EARS-04; FR-12)
"Once indexing completes" vs "once both complete".
Fix: Align the EARS file to "Once listing and indexing complete".

Carry-over (not counted, Downstream usability): residual synonyms ("results", "folder access", "search box", "search results area", "preview panel"). Fix as in `review-rubric-v1.md`; add **preview panel** to the Glossary.

## Prior findings status
14 critical and high findings from `review-rubric-v1.md`, `review-ears-v1.md`, and `review-adversarial-general.md`: 10 resolved, 2 partially resolved (C-2 → N-6, H-5 → N-1), 2 accepted-risk (C-1 DOCX, H-6 cloud-synced folders), 0 open. See `review-rubric.md` for the full table.

## Mechanical notes
- New rubric finding counts: critical 0, high 0, medium 6, low 7; one low carry-over not counted.
- IDs are contiguous and unique; all cross-references resolve. SM-1 "Validates FR-8 to FR-21" omits FR-22 (defensible).
- Assumptions Index roundtrip is clean. FR-21 and FR-22 introduce exact UI strings without assumption tags (adversarial L-6 pattern).
- EARS traceability: §0 and the EARS file agree; FR-20's narrowing of EARS-02 to "with a folder selected" is still undocumented (EARS F-07, medium).
- Process fixes since v1: no OQ blocks a phase; FR-19 wording fixed; NFR-3 storage list extended; NFR-4 workload defined; mixed-query gap explicit.
- Adversarial medium and low carry-overs still open: M-1, M-2, M-4, M-5, M-7, M-9, M-10, M-11, M-12, M-13, L-2, L-4. Partly addressed: M-6, M-8.
- OQ-3, OQ-6, OQ-8 are Decided/Resolved but still listed among open questions (cosmetic).
- Required sections are all present.

## Reviewer files
- `review-rubric.md` (primary, re-validation)
- `review-ears.md` (EARS re-check v2)
- `review-adversarial-general.md` (previous PRD version; tracked via the rubric's prior-findings table, not a current reviewer)
- `review-rubric-v1.md`, `review-ears-v1.md` (superseded)
