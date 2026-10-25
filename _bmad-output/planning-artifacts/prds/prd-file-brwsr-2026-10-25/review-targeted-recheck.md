---
title: "Targeted re-check of PRD fixes: PRD Find"
reviewed: 2026-10-25
inputs:
  - _bmad-output/planning-artifacts/prds/prd-file-brwsr-2026-10-25/prd.md
  - docs/product/earsRequirments
  - review-rubric.md (findings N-1 to N-6)
  - review-ears.md (findings N-01, F-06, F-07)
scope: only the items listed below, plus a scan for new contradictions and an MVP-scope check. prd.md and the EARS file were not edited.
---

# Targeted re-check: PRD "Find"

## Summary

| # | Finding | Status |
|---|---|---|
| 1 | EARS N-01: EARS-03 / EARS-04 overlap | **Resolved** (two low wording notes) |
| 2 | Rubric N-1: substring floor and SM-3 competing fixtures | **Resolved** as scoped (SM-C1 not extended; low residual). The floor creates a new EARS-01 deviation (NEW-1). |
| 3 | Rubric N-2: SM-8 separate from SM-1 | **Partially resolved** (whether the seeded folder is the MyFiles folder is still unstated) |
| 4 | Rubric N-3: FR-22 behaviour during a rescan | **Resolved** |
| 5 | Rubric N-4: FR-10 prefix length and `reprot`/`Reports` | **Resolved** (no negative example; low residual) |
| 6 | Rubric N-5: NFR-1 / NFR-5 scope, OQ-1 PDF-tab check | **Resolved** in the NFRs and OQ-1, but FR-21 still promises "no network request" (NEW-2) |
| 7 | Rubric N-6: UJ-1 resolution, SM-1 stop, OQ-1 locate method | **Resolved** |
| 8 | EARS F-06 (preview panel) and F-07 (FR-20 EARS-02 test) | **Resolved** (both) |

**New findings:** 0 critical, 0 high, 2 medium (NEW-1, NEW-2), 4 low (NEW-3 to NEW-6).

**MVP scope (§7.1 / §7.2) unchanged:** yes.

---

## 1. EARS N-01: EARS-03 rev. 2 vs EARS-04 (Resolved)

**Texts in force.**
- EARS-03 rev. 2: "If no files match a non-empty search term, **listing has completed**, and indexing has completed or was cancelled, then the system shall display a 'No matching files found' message".
- EARS-04: "**While listing or indexing is in progress**, if no files match a non-empty search term, the system shall display a 'No matches yet. Still searching…' message".
- FR-12 bullet 3: "If indexing was cancelled (FR-13) after listing completed, and nothing matches, Find shows 'No matching files found' … While listing is still running, EARS-04 applies even if indexing was cancelled. A rescan (FR-22) clears the cancelled state."
- FR-22: "During a rescan, the result list shows *still updating*, and EARS-04 applies."

**State walk** (no match, non-empty query, no filters hiding matches):

| Listing | Indexing | EARS-03 rev. 2 | EARS-04 | Result |
|---|---|---|---|---|
| running | running | false (listing not complete) | true | EARS-04 only |
| running | done | false | true (listing in progress) | EARS-04 only. In practice unreachable: indexing can't finish before listing has reached every file. |
| running | cancelled | false | true | EARS-04 only. This was the N-01 overlap; FR-12 now states it explicitly. |
| done | running | false (indexing neither done nor cancelled) | true | EARS-04 only |
| done | done | true | false | EARS-03 only |
| done | cancelled | true | false (nothing in progress) | EARS-03 only, plus FR-12's second line |
| rescan: listing running (any indexing state, cancel cleared) | | false (current listing not complete) | true | EARS-04 only (FR-22) |
| rescan: listing done, indexing running | | false | true | EARS-04 only |
| rescan: listing done, indexing done / cancelled again | | true | false | EARS-03 only |

No state makes both true. In every reachable state exactly one applies, so there is no gap either. The one edge case is a folder above 50,000 file entries, where NFR-4 says it is "not supported" without saying whether listing completes. That is the pre-existing adversarial M-2 and is not new.

**FR-12 vs the EARS file.**
- EARS-03 message, area, fixture (`report.pdf`, `photo.jpg`) and query (`xyz123`) match exactly.
- EARS-04 message "No matches yet. Still searching…" matches exactly, in FR-12, UJ-1, and §0.
- The acceptance-test wording is paraphrased, not word for word (see NEW-6):
  - EARS-03 test. EARS: "Wait until listing completes and indexing completes (or cancel indexing after listing completes)". FR-12: "after listing completes and indexing completes or is cancelled". FR-12 also covers a cancel *during* listing followed by listing completing. That state is allowed by the rev. 2 statement, so the two don't conflict.
  - EARS-04 test. EARS: "with indexing still in progress … Once indexing completes". FR-12: "while listing or indexing is in progress … once both complete". This is the carry-over EARS N-06 (low). FR-12 is the more precise of the two.

**Traceability.**
- The EARS file's Revision History has a rev. 2 entry with the reason, rev. 1's statement verbatim ("superseded, kept for traceability"), the rev. 1 entry, and the original EARS-03 statement and acceptance test verbatim.
- §0 table, EARS-03 row: "Revised (rev. 2, 2026-10-25). The original and rev. 1 are kept in the EARS file's revision history". EARS-04 row: "New, derived from EARS-03 rev. 1", which matches the EARS-04 heading "(new, derived from EARS-03 rev. 1; unchanged in rev. 2)".
- Residual (low, NEW-6): only rev. 1's *statement* is kept. Rev. 1's acceptance test is not in the history. The original's test is.

## 2. Rubric N-1: substring floor and SM-3 (Resolved as scoped)

- **Floor.** FR-9: "A query word shorter than 3 characters matches only as an exact token or a token beginning, never as a substring. So `re` does not match `are.pdf`." This sets a 3-character minimum for substring matching. The rubric's own suggestion was stricter ("3 or fewer"), so `tax` → `syntax.md` is still a **substring** match. That is acceptable under the floor the user chose.
- **Examples vs the floor.** All are consistent:
  - FR-9: `port` (4) → `report.pdf` substring; `report` (6) → `annualreport2025.pdf` substring; `proj rep` → partial (`rep` is 3 characters and is a token beginning, so the floor doesn't apply); `q3 report` → exact (`q3` is a whole token).
  - FR-11: `report` → `annualreport2025.pdf` (substring) ranks below `Q3_Reports.pdf` (partial); `project reprot` → typo; `budget` → content above `budgte_2024.xlsx` typo. None of these rely on a substring word under 3 characters.
  - §3 Match kind example `port` in `report.pdf`: consistent.
- **SM-3.** "The fixture set includes queries where other files' names contain the query words, so content matches compete with substring matches." Done.
- **Residuals.**
  - SM-C1 still measures only "the SM-2 fixture set". The rubric's fix also asked to extend it to the SM-3 competition (low).
  - The floor is stated only in FR-9. §3 Name match ("text anywhere in the name") and §3 Match kind **substring** don't mention it (NEW-5, low).
  - The floor narrows EARS-01, and that is not logged (NEW-1, medium).

## 3. Rubric N-2: SM-8 vs SM-1 (Partially resolved)

- SM-8: "Measured as a separate task before SM-1: each user follows the MyFiles guide without help." Done.
- SM-1 Timing: "the clock starts in the start state, with MyFiles already set up (SM-8)". Done.
- **Still open.** SM-1 Protocol says "the seeded folder sits in a location the operating system's search indexes". It doesn't say whether the seeded folder *is* the MyFiles folder.
  - If it is, `C:\MyFiles` is outside Windows Search's default indexed locations, so the protocol must add it to the OS index for the comparison to be fair.
  - If it isn't, "MyFiles already set up" has no effect on SM-1.
  - The rubric's fix asked for this to be stated. Severity: low (it affects test-protocol fairness, not the build).

## 4. Rubric N-3: FR-22 during a rescan (Resolved)

FR-22 now states one behaviour: "During a rescan, the current folder tree and result list stay visible and usable. File entries are replaced as listing reaches them, and removed file entries disappear when listing completes. The content index is replaced the same way as indexing reaches each file."
- "EARS-04 applies" is now consistent: EARS-04's own condition requires no match, so it shows only when the kept result list is empty.
- Per-file replacement means the old and new content indexes are never held in full at once, so the NFR-4 200 MB limit isn't doubled.
- The query, filters, and selected file entry are kept. The cancelled state is cleared (FR-12).
- The low carry-over N-10 ("Listed at HH:MM" semantics; a file deleted between rescans) was not in scope and is unchanged.

## 5. Rubric N-4: FR-10 prefix length (Resolved)

- FR-10: "A token beginning is compared at the query word's length, or one letter longer or shorter. So `reprot` is compared with `Report` from `Reports`."
- Check: for `reprot` (6 letters, 1 edit allowed), the prefixes of `Reports` are `Repor`, `Report`, and `Reports`. OSA(`reprot`, `report`) = 1 (a swap), so it matches. The consequence "`reprot` matches `Reports.pdf`, against the token beginning `Report`" holds.
- The rubric's test questions are now decidable:
  - `budgte` → `budgeting` is a typo match (prefix `budget`, 1 swap).
  - `from` → `format` is a typo match (prefix `form`, 1 swap).
  - `repo` vs `rep` gets no tolerance at 3 letters.
- Residual (low): there is no negative example. The `from` → `format` case is a false-match class SM-C1 will see, and it would make a useful explicit example.

## 6. Rubric N-5: NFR-1 / NFR-5 scope and the OQ-1 PDF-tab check (Resolved, with NEW-2)

- NFR-1: "After the initial page load, **Find itself** makes zero network requests … Opening a file in a tab (FR-21) uses a local in-browser link, not the network."
- NFR-5: "Content shown inside Find is inert … Files opened in a new tab (FR-21) are displayed by the browser's own built-in viewer, which is outside Find's control. Text files still open as plain text (FR-21). Whether opening a PDF in a tab makes any network request is verified in the OQ-1 spike."
- OQ-1: "Does opening a PDF in a new tab (FR-21) make any network request?"
- The scope is now honest. However:
  - FR-21 still asserts "opens a PDF, image, or text file in a new browser tab, read-only, **with no network request**". That is an unverified guarantee NFR-5 now disclaims (NEW-2, medium).
  - SM-6 counts "zero network requests" for the whole session, which is not scoped to "Find itself" (NEW-4, low).

## 7. Rubric N-6: hand-off (Resolved)

- UJ-1 Resolution: "clicks **Copy location** (`MyFiles/Clients/Acme/Project_Report_Sep.pdf`), opens his MyFiles folder in File Explorer, follows the path to the file, and opens it in Acrobat." He no longer pastes a relative path.
- SM-1 Timing stop: "when the correct file is open in a Find tab, or located in File Explorer or Finder." This no longer depends on opening the file in a native app.
- OQ-1: "What is the most reliable way to locate a file in File Explorer and Finder from a copied location?"
- FR-21 hint ("Use **Copy location** to find it in File Explorer or Finder") is consistent with all of these.
- Minor: what counts as "located" (visible, or selected) isn't defined for the moderator. Not counted.

## 8. EARS F-06 and F-07 (Resolved)

- **F-06.** §3 Preview: "It appears in the preview panel; EARS-02 uses that name." FR-20 uses "the preview panel shows". The term is now anchored in the Glossary. It is a mention inside **Preview** rather than its own entry, which is sufficient.
- **F-07.** FR-20: "in a selected folder containing files, with no selected file entry, the preview panel shows 'Select a file to preview'." This now mirrors the EARS-02 test exactly. The EARS file's EARS-02 is unchanged against HEAD; its test input was already "A selected folder containing files", so the PRD moved to the EARS text, not the other way round. §0's "Unchanged" for EARS-02 is therefore accurate. The added "In the start state, no preview is shown (§3 Start state)" is consistent: EARS-02 places the message "in the preview panel", which doesn't exist in the start state.

---

## New findings

- **NEW-1 [MEDIUM] The substring floor narrows EARS-01, but traceability still says "Unchanged" and FR-9 still claims full satisfaction.**
  - Location: FR-9 last two bullets; §0 EARS-01 row; EARS-01.
  - EARS-01: "display all files … whose names **contain** the search term".
  - FR-9: "`port` matches `report.pdf` … This satisfies EARS-01's 'names contain the search term'", followed by "A query word shorter than 3 characters matches only as an exact token or a token beginning, never as a substring. So `re` does not match `are.pdf`."
  - Under EARS-01, `are.pdf` contains `re` and must be shown. The PRD now deliberately omits it, and the §0 row "EARS-01 … Unchanged … FR-8, FR-9 (substring match kind)" doesn't log the deviation.
  - The EARS-01 acceptance test (`report`) still passes.
  - Fix: add a deviation note to the §0 EARS-01 row ("query words under 3 characters do not substring-match, per FR-9"), qualify FR-9's "This satisfies EARS-01" sentence, and either log it in the EARS revision history or accept it as a PRD-level narrowing.
- **NEW-2 [MEDIUM] FR-21 still guarantees "no network request" for PDFs opened in a tab, which NFR-5 and OQ-1 now treat as unverified.**
  - Location: FR-21 bullet 1; NFR-5; OQ-1.
  - FR-21: "opens a PDF, image, or text file in a new browser tab, read-only, with no network request". NFR-5: displayed "by the browser's own built-in viewer, which is outside Find's control … Whether opening a PDF in a tab makes any network request is verified in the OQ-1 spike."
  - A testable consequence asserts what the spike has yet to verify. A story written from FR-21 will promise it.
  - Fix: reword FR-21 to "Find makes no network request to open it (NFR-1)", or defer the PDF clause to the OQ-1 result.
- **NEW-3 [LOW] FR-12 bullet 3 can be read as covering only a cancel made after listing completed.**
  - "If indexing was cancelled (FR-13) after listing completed" reads as *when the cancel happened*. A cancel made during listing, after which listing completes, satisfies EARS-03 rev. 2. But it is unclear whether the second line ("Content search was cancelled. Some files weren't searched.") appears.
  - Fix: "If listing has completed, indexing was cancelled, and nothing matches…".
- **NEW-4 [LOW] SM-6 is not scoped like NFR-1.**
  - SM-6 counts "zero network requests" for the whole session. NFR-1 now limits the guarantee to "Find itself", and NFR-5 excludes the browser's own viewer.
  - A PDF opened in a tab, or a link the user clicks inside it, could fail SM-6 without breaking NFR-1.
  - Fix: "zero network requests made by Find".
- **NEW-5 [LOW] The substring floor isn't reflected in the Glossary.**
  - §3 Name match ("text anywhere in the name") and §3 Match kind **substring** ("every query word appears somewhere in the name") don't mention the 3-character minimum. The Glossary claims to define each term once, so readers of §3 alone will miss the floor.
  - Fix: add "(query words of 3 or more characters, FR-9)" to the substring definition.
- **NEW-6 [LOW] EARS traceability details.**
  - FR-12's acceptance tests paraphrase the EARS tests rather than reproducing them word for word (see §1). The results are equivalent.
  - Rev. 1's acceptance test is not preserved in the EARS Revision History; only its statement is.

**Checked, no new contradiction:**
- §0 traceability table (EARS-02, EARS-03, and EARS-04 rows agree with the EARS file).
- FR-21 apart from NEW-2: the hint text agrees with UJ-1 and SM-1.
- Assumptions Index: every inline `[ASSUMPTION]` tag (UJ-1, §3 Type, FR-1, FR-5, FR-10, FR-13, FR-14, FR-16, FR-18, FR-20, NFR-4, NFR-6, §7.2, SM-1, SM-8, SM-C1) has an entry, and every entry has a tag. The new FR-9 floor and the FR-10 ±1 rule are stated as decisions, not assumptions, which is consistent with the index's FR-10 entry covering only the edit limits.
- §3 Rank and the FR-11 examples.

## MVP scope

**Unchanged.**
- §7.1 still lists FR-1 to FR-22 in the same groupings.
- §7.2 still holds the same items: DOCX/XLSX content with its `[NOTE FOR PM]`, version clusters, duplicate detection, desktop apps and full paths, cloud-only detection, automatic change detection, split name/content queries, the list/thumbnail switch, recent-usage ranking, typo tolerance in content and non-English text, day-first dates, browsers, multiple folders, and conversational search.
- These match the scope described in `review-rubric.md` and the memlog decision ("MVP scope unchanged").
- The edits under review touched FR-9, FR-10, FR-12, FR-20, FR-22, NFR-1, NFR-5, UJ-1, SM-1, SM-3, SM-8, OQ-1, and the Glossary. None of them adds or removes a capability.
