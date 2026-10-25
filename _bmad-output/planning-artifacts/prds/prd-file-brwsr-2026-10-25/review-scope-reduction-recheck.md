---
title: "Targeted re-check: filename-only scope reduction"
date: 2026-10-25
inputs:
  - prd.md (working tree, after the 2026-10-25 scope reduction)
  - docs/product/earsRequirments (working tree)
  - briefs/brief-file-brwsr-2026-10-25/brief.md
  - .memlog.md (last 8 entries)
  - baseline for comparison: prd.md at commit f0f6934 (git HEAD)
edits_made: none (review only)
---

# Targeted re-check: filename-only scope reduction

**Verdict:** The reduction is sound, and the MVP core holds together. EARS-03 rev. 3 and EARS-04 rev. 1 cannot overlap, and every EARS acceptance test appears in an FR. Section 11, however, does not keep the deferred text "as last approved", as it claims to. The status table mislabels four changed requirements as unchanged. FR-12 also leaves several states undefined, including listing errors.

**Counts:** critical 0 · high 1 · medium 6 · low 9

Line numbers refer to the current working-tree `prd.md` unless another file is named.

---

## Check 1: Leftover deferred references in §0–§10

**Result: clean.** Every mention of tree, preview, rank, match kind, token, typo, content, indexing, name date, rescan, opening in a tab, size, and selected file entry is either marked deferred or is not about the feature:

- Marked deferred: L41, L51, L78, L182, L210, L223, L255, L275, L293–L299, L329, L339, L340, L342.
- Not about the feature:
  - L260 (NFR-2): "size" is a tester's integrity check in the operating system, not a displayed detail.
  - L274: "no always-on index" is a non-goal.
  - L306: "Ranking by recent usage" is an impossibility note.
  - L315: the operating system's search "indexes".
  - L338: "folder sizes".

Two wording nits are recorded as L-8.

## Check 2: MVP FR consistency and testability

- **FR-9 and splitting at `.`:** this does not break extensions. Any name that contains the literal query also contains every piece of it, so the result list is always a superset of what EARS-01 requires. `report.pdf` → {`report`, `pdf`} still finds `report.pdf`. The cost is extra noise, such as `pdf_report.docx`, and a single-character piece like `v1.2` → `2` matches any name with a "2" in it. See L-2. The real gap is joined versus separated words. See M-3.
- **FR-12 precedence:** the core no-match states are covered, but several states are left undefined. See M-4 and M-5.
- **Empty-query prompt versus the Glossary:** these are consistent. The Glossary (L122) makes a query of only separators an empty query. The start state (L103) has no result list, and the FR-12 prompt (L220) applies only after a folder is selected. EARS-01 read literally is a separate issue. See M-6.
- **FR-6, FR-15, FR-17, FR-21:** testable as written. Minor gaps are in L-1 and L-4.

## Check 3: Traceability

- **§0 status table:** every FR from FR-1 to FR-22 appears exactly once (11 MVP, 11 deferred). NFR-1 to NFR-8 each appear once. However, four "unchanged" labels are wrong. See M-1.
- **§11:** every ID is preserved, and the core substance of each deferred FR is kept. Many testable details are lost, though, and the heading claims the text is verbatim. See H-1.
- **EARS table and EARS file:** these match. EARS-01 is unchanged, EARS-02 is deferred with its text unchanged, EARS-03 is at rev. 3, and EARS-04 is at rev. 1.
- **EARS revision history:** every earlier version is kept: the original EARS-03, rev. 1, rev. 2 (with its test), and the original EARS-04. There is some housekeeping to do. See L-6.

## Check 4: EARS-03 rev. 3 and EARS-04 rev. 1

- **Mutual exclusion:** they cannot both apply. EARS-03 requires "listing has completed", and EARS-04 requires "listing is in progress". Listing runs once per session (Glossary L109), Rescan is deferred, and choosing a new folder starts a new session. There is no state where both hold.
- **Gap:** there is a third state where *neither* holds: listing stops without completing. See M-5.
- **Acceptance tests reproduced:**
  - EARS-01 → FR-8 L193, verbatim in substance.
  - EARS-03 → FR-12 L217.
  - EARS-04 → FR-12 L218, including the change to "No matching files found".
  - EARS-02 → §11 FR-20 L436.

## Check 5: Metrics

- SM-4, SM-6, and SM-9 are valid for the MVP.
- SM-1 is valid. Its cues and the FRs it validates match the MVP.
- SM-2 can fail its own 100% target. See M-3.
- SM-C1 is a reasonable counter-metric, and its threshold is correctly tagged and indexed. Its fixture still needs definition. See L-2.
- No deferred metric (SM-3, 5, 7, 8, C2, C3) is cited as active anywhere in §0–§10.
- One NFR-4 target no longer has a metric. See L-3.

## Check 6: Brief consistency

- The brief matches the PRD on: the MVP list, alphabetical order, the type filter notice, Copy location, the MyFiles hint, and the out-of-scope list.
- It differs on: "separators ignored" (M-3), "basic error handling" (M-5), and several low-severity points (L-7).

---

## Findings

### H-1 (high): §11 claims the deferred text is kept "as last approved", but it is condensed, and approved testable criteria were lost

- **Location:** §11 intro (L362), §11.1 (L364–L374), §11.2 (L376–L397), §11.3 (L399–L448).
- **Problem:** L362 says deferred requirements "keep their IDs and text **as last approved**". In fact they are paraphrased summaries. Compared with f0f6934, these items are lost:
  - **Match kinds:** the §3 match-kind definitions (exact = equals a whole token; partial = a token beginning; substring = only inside a token; "a file entry that is both… gets the better of the two kinds"). §11.1 lists only the kind names.
  - **Rank:** the rule for an empty query ("most recent modified date first").
  - **FR-12:** the Out of Scope note (a query split between name and content matches nothing). The cancel rule, "holds whether the cancel happened during or after listing", and "a rescan clears the cancelled state".
  - **FR-13:** "Indexing starts automatically"; progress counted against "text files and PDFs"; "name search and the folder tree keep working after a cancel, and the result list states that content search is incomplete".
  - **FR-14:** the snippet is "around the first match, with the matched words highlighted"; "[ASSUMPTION: architecture should treat this rule as fixed]".
  - **FR-16:** the 10 MB limit is "per file"; the NFR-4 cross-reference for "index limit reached".
  - **FR-18:** an inclusive range; either end may be left empty; the `invoice_2025-12.pdf` / 10 Jan 2026 example; the year-only `report_2024.pdf` example; the example of a day-first format being rejected.
  - **FR-20:** "In the start state, no preview is shown"; "For a PDF, it opens on that page".
  - **FR-21:** text files, including HTML, never run as a page; the exact hint for type *other*; "Find itself makes no network request… verified in OQ-1".
  - **FR-22:** the deleted-entries timing; the content index is replaced progressively; "If the browser no longer grants access, Find returns to the start state with the FR-2 hint"; the Out of Scope note (no automatic change detection; a rescan is a full rebuild).
  - **FR-5 and FR-7:** "Expanding a folder shows its subfolders and file entries"; sorting "in an expanded folder of the folder tree".
  - **FR-10:** the `project reprot` → **typo** example.
  - **Deferred metrics:** SM-3, SM-5, SM-7, SM-8, SM-C2, and SM-C3 are cut to titles, losing their targets. Examples: SM-5's 60 s with no freeze over 100 ms; SM-8's "4 of 5 users… first try"; SM-C3's `ticket_20251` example. The pre-reduction texts of SM-1, SM-2, and SM-C1 are not kept anywhere.
  - **Later edits:** the working-tree edits made after f0f6934 are not in git, so the only copy of the true last-approved version is gone. Examples: the §7.2 "Full-fidelity PDF preview" entry with its AD-14/AD-21 references.
- **Fix (choose one):**
  - **(a)** Restore the deferred FR, Glossary, and metric texts verbatim into §11 from f0f6934, and re-add the post-f0f6934 FR-20 limitation text.
  - **(b)** Reword L362 to "keep their IDs and substance; the full approved text is in commit f0f6934 (`prd.md`)". Then commit the current working tree, so that the reduction itself is a recorded change against that baseline.

  Option (a) is better if these items are expected to return within the project. Option (b) is acceptable for a portfolio trail.

### M-1 (medium): The §0 status table labels changed requirements as "unchanged", and some removed parts are missing from §11.2

- **Location:** §0 L32 to L34; §11.2 (L376 to L397).
- **Problem:** These were edited during the reduction:
  - **FR-2:** lost "The hint links to the **Set up MyFiles** guide".
  - **FR-3:** "discards the previous folder tree, content index, query, filters, and selected file entry" became "file entries, query, and filters".
  - **FR-8:** gained the result-order consequence (L196).
  - **NFR-3:** lost "file content" (see M-2).

  In addition, §11.2 does not record removed parts of these requirements:
  - FR-2: the guide link.
  - FR-3: the tree, index, and selection.
  - NFR-1: "including the PDF library" and the in-tab local link.
  - NFR-6: the folder tree, the preview, and the FR-21 actions.
  - NFR-8: ranking, name-date detection, and indexing rules.

  FR-15 is listed in §11.2, but it has no "*Deferred from…*" note in §4.3, unlike FR-6, FR-9, FR-12, and FR-21.
- **Fix:**
  - Move FR-2, FR-3, and NFR-3 to "MVP, simplified". Label FR-8 "MVP, extended (result order)", or move it to "simplified".
  - Add the missing removed parts to §11.2.
  - Add a "*Deferred from the original FR-15 (§11): indexing.*" line after L231.

### M-2 (medium): NFR-3 no longer covers file content, which is narrower than the EARS project constraint

- **Location:** NFR-3 (L261); EARS file, Project Constraints, L51 ("No persistent storage of file contents or metadata").
- **Problem:** "file content" was removed from NFR-3 because the MVP reads no content. But the EARS constraint still names file contents, and the requirement is labelled "unchanged". A future reintroduction of content search would start from a weaker guarantee.
- **Fix:** Restore "file content" to NFR-3's list. It costs nothing in the MVP and keeps the requirement aligned with the EARS constraint.

### M-3 (medium): Joined and separated words don't match, which contradicts the brief's "separators ignored" and can fail SM-2's 100% target

- **Location:**
  - PRD: FR-9 (L200 to L208); SM-2 (L319).
  - Brief: L69 ("with separators ignored").
- **Problem:** Matching runs on the raw name. The query `projectreport` (or `ProjectReport`) does **not** match `Project_Report_Q3.docx`, because the literal piece `projectreport` is not in the name. Only the opposite direction works: `project report` matches `ProjectReport.txt`. SM-2's fixture set explicitly includes "different separators", so a natural fixture of this kind fails a 100% target. The brief also promises that separators are ignored.
- **Fix (choose one):**
  - **(a)** Add an FR-9 rule: each query word is also compared against the name with separators removed (spaces, `_`, `-`, `.`). Add the consequence "`projectreport` matches `Project_Report_Q3.docx`".
  - **(b)** Narrow SM-2's fixture description and the brief to "different separators *between* words". Add a negative example to FR-9 that shows `projectreport` does not match `Project_Report_Q3.docx`.

### M-4 (medium): FR-12 leaves several states undefined

- **Location:** FR-12 (L212 to L221); Glossary L122 to L126.
- **Problem:**
  - **(a) Empty query with an active filter (L221):** it is not defined whether "Filters hide N matches" appears, or what N counts. With an empty query, every file entry would count as a "match", giving a notice like "Filters hide 9,412 matches".
  - **(b) A filter is active, but the query matches nothing at all (N = 0):** "When the type filter hides matches" implies N ≥ 1, but it doesn't say so. A naive implementation could show "Filters hide 0 matches" instead of EARS-03 or EARS-04.
  - **(c) An empty folder** (no file entries, or only subfolders): after listing completes, the empty-query prompt "Type part of a file name to search" is misleading. The brief promises "clear empty states".
  - **(d) "Takes precedence" wording (L219):** by the Glossary, a filtered-out match is still a name match. So EARS-03 ("no files match") and EARS-04 cannot co-occur with the notice. "Precedence" suggests a conflict that does not exist.
- **Fix:**
  - **(a)** State the rule. Suggested: with an empty query, no "Filters hide" notice; the type filter just lists what passes.
  - **(b)** Add "The notice appears only when N ≥ 1; otherwise EARS-03 or EARS-04 applies."
  - **(c)** Add an empty-folder state, for example "This folder has no files", shown once listing completes with zero file entries. Tag it `[ASSUMPTION: wording]` and index it in §10.
  - **(d)** Reword it as: "Because hidden matches are still name matches, EARS-03 and EARS-04 don't apply; Find shows this notice instead."

  Optionally, add a small state table to FR-12 with the columns: query empty?, filter?, listing state, name matches, visible, message.

### M-5 (medium): Listing that stops without completing is unspecified, so neither EARS-03 nor EARS-04 applies and there are no "basic errors"

- **Location:**
  - PRD: FR-12; FR-15; NFR-4 (L264, the "not supported… Find says so" limit above 50,000 file entries); Glossary "Listing" (L109); §1.1 Principle 1 (L57).
  - Brief: L95 ("Clear empty states and basic error handling").
- **Problem:** Listing can end without completing:
  - read access is revoked mid-session;
  - a subfolder cannot be read;
  - the folder exceeds the 50,000-entry limit.

  None of these cases is specified. EARS-04 needs "in progress" and EARS-03 needs "completed", so no message applies, and the *still updating* indicator (FR-15) has no end condition. A silently skipped subfolder would also break Principle 1 ("never silently removes a file entry"). The approved FR-22 had covered access loss ("returns to the start state with the FR-2 hint"), but that requirement is now deferred.
- **Fix:** Add a consequence to FR-12 (or a new FR-12 bullet group, "Listing problems"):
  - **Unreadable subfolders** are skipped. Listing continues, and once it completes Find shows "N folders couldn't be read". Listing then counts as completed for EARS-03.
  - **Lost access** returns Find to the start state with the FR-2 hint.
  - **Over the 50,000 limit:** listing stops, Find says the folder is too large, and suggests a subfolder.

  Define in the Glossary that "listing has completed" also covers a listing that ended with skipped subfolders.

### M-6 (medium): The §0 EARS-01 row says "fully satisfied", but a query of only separators is handled differently from EARS-01's literal reading

- **Location:** §0 L40; Glossary L122; FR-12 L220.
- **Problem:** EARS-01: "When the user enters a search term, the system shall display all files… whose names contain the search term." Under the Glossary, a term made only of separators, such as `-`, `_`, or `.`, is an empty query. Find then shows the prompt instead of the files whose names contain `-`. That is a narrowing, the same class of issue as NEW-1 in the previous re-check, which the user chose to record explicitly. The row also doesn't mention the deliberate broadening: word splitting and any-order matching return files that don't contain the literal term, such as `final-report.docx` for `report-final`. The EARS-01 test still passes.
- **Fix:** Change the row to: "Unchanged. Satisfied as a superset (query words in any order). One deliberate narrowing: a term made only of separators is treated as an empty query (Glossary, FR-12)." Add the same note to FR-9.

### L-1 (low): The result order and case-insensitive matching need an exact comparator

- **Location:** FR-8 L196; FR-9 L200; Glossary L126.
- **Problem:**
  - "Alphabetical, ignoring case" doesn't define the collation: code point or locale, numeric (`file2` vs `file10`), or accents. It also has no final tie-break for names that differ only in case at the same location, which can happen on Linux.
  - Unicode normalization (NFC/NFD, which matters for macOS names) and case folding are not defined for matching.
  - NFR-8's unit tests need these to be deterministic.
- **Fix:** Specify, for example, "`Intl.Collator('en', {sensitivity: 'base'})` on name, then location, then a code-point compare as the final tie-break". Specify "names and queries are NFC-normalized and lower-cased with locale-independent case folding". Or delegate both explicitly to architecture.

### L-2 (low): The SM-C1 fixture is undefined, and short query words are not covered

- **Location:** SM-C1 L327; FR-9.
- **Problem:** The 3-character floor was removed. The query words `a`, `e`, and `2` now match most names, and the result list is noisiest while the user types the first letter. A median result count of 20 or fewer means nothing without the size and makeup of the fixture folder.
- **Fix:** In SM-C1, state the size of the fixture folder (for example, 1,000 file entries) and include 1–2-character query words. Optionally, note that a 1-character query is expected to be noisy, and that this is acceptable.

### L-3 (low): The NFR-4 100 ms freeze limit is not validated by any active metric

- **Location:** NFR-4 L265; §8.
- **Problem:** SM-5 (deferred) carried the "no freeze over 100 ms" check. SM-4 covers only result latency.
- **Fix:** Add to SM-4: "…and no main-thread task over 100 ms during listing of 10,000 file entries".

### L-4 (low): FR-21 does not define what happens when a copy fails

- **Location:** FR-21 L252.
- **Problem:** The clipboard write can be refused, for example if the page is not focused or permission is denied.
- **Fix:** Add a consequence: "If copying fails, Find shows the location as selectable text with 'Couldn't copy, select and copy it manually'."

### L-5 (low): The FR-1 hint lost the "not cloud-synced" guidance that the Glossary and OQ-9 rely on

- **Location:** FR-1 L142; Glossary L108; OQ-9 L345.
- **Problem:** The Glossary defines MyFiles as not cloud-synced, but the MVP hint shown to users never says so.
- **Fix:** Add "in a folder that isn't synced to the cloud" to the hint text. Its wording is already tagged `[ASSUMPTION]`.

### L-6 (low): EARS file housekeeping

- **Location:** `docs/product/earsRequirments` L1, L54 to L76.
- **Problem:**
  - **Title:** "Three EARS Requirements" while the file has four.
  - **Entry dates:** all history entries are dated 2026-10-25, so their order is shown only by position.
  - **Stale reference:** the rev. 2 entry cites "a rescan clears the cancelled state (PRD FR-12, FR-22)". FR-22 is now deferred, and FR-12 no longer contains that rule.
  - **Missing test:** the rev. 1 acceptance test is not quoted. This was accepted earlier and is noted again only for completeness.
- **Fix:**
  - Retitle to "EARS Requirements".
  - Add "(rev. 1)", "(rev. 2)", and "(rev. 3)" sequence labels to the dated headings.
  - Append "(both now deferred, PRD §11)" to the rev. 2 entry's FR reference.

### L-7 (low): Minor brief inconsistencies

- **Location:** brief L48 to L50, L81, L85.
- **Problem:**
  - **Secondary users:** "developers inspecting folder structures" and "all of last year's invoices at tax time" depend on the deferred folder tree and date filter.
  - **Success criterion:** the brief describes a "one-week trial", while PRD SM-1 is a moderated session.
  - **BMAD trail:** the brief's trail includes "implementation", but SM-9's does not.
- **Fix:**
  - Qualify the secondary users with "(after the MVP)".
  - Align the success criterion with SM-1, or note that the PRD supersedes it (the brief's L10 already says that the PRD wins).

### L-8 (low): Wording nits in §0–§10

- **Location:** L186; L260.
- **Problem:**
  - L186, "Search forgives missing words…", can be read as matching *any* query word, which contradicts FR-9's every-word rule.
  - L260, the NFR-2 verify step, checks "size", which the MVP no longer displays. This is valid as an operating-system check, but it could be read as a leftover.
- **Fix:**
  - L186: change to "…forgives a partial name, different separators, wrong case, and word order."
  - L260: change to "…every file's modified date and size, as reported by the operating system, are unchanged".

### L-9 (low): Some terms are used without Glossary entries

- **Location:** §3; FR-15 L230; FR-1 L142; L122.
- **Problem:** "still updating", "MyFiles hint", "query word", and "empty query" are used as defined terms. They are not separate Glossary entries; "query word" and "empty query" are defined only inline under Query.
- **Fix:** Add short entries for **Still updating** (shown while listing runs, FR-15), **MyFiles hint** (FR-1), and **Empty query** (no query words after splitting).
