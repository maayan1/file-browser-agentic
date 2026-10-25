# Validation Report — PRD: Find

- **PRD:** `_bmad-output/planning-artifacts/prds/prd-file-brwsr-2026-10-25/prd.md (+ addendum.md)`
- **Rubric:** `.claude/skills/bmad-prd/assets/prd-validation-checklist.md`
- **Run at:** 2026-10-25T16:41:28+02:00
- **Grade:** Fair

## Overall verdict

This is a strong, build-ready PRD. The thesis is explicit and tested by the metrics (SM-1 now forces content and name-date tasks, SM-3 isolates content search). The Glossary is disciplined, scope cuts are honest, and every high and medium finding from the pre-final review is resolved. What remains is concentrated in the matching rules that stories and tests will lean on hardest. FR-10's headline acceptance example (`reprot` → `report`) fails under standard edit distance with the stated 1-edit limit, so "edit" must be defined. How a query is split into query words is undefined. The "exact" definition and EARS-01 use substring language that FR-9's token rules don't honor. Separately, two product bets go unmeasured: the MyFiles strategy rests on an unverified platform premise, and queries that mix a name cue with a content cue, which is the core job, match nothing. None of this blocks UX. The matching items should be settled before stories are written.

The grade above (Fair) is derived from the rubric alone: no thin or broken dimensions, 0 critical findings and 1 high. The two additional reviewers surfaced considerably more. The EARS-conformance reviewer found all three EARS requirements satisfied with caveats, with 3 high findings: token-beginning matching narrows EARS-01's "names contain the search term", the undefined query-splitting rule means `report-final` may not find `report-final.docx`, and FR-12 replaces EARS-03's message while indexing runs and adds a precondition to its acceptance test. The adversarial reviewer found 2 critical and 8 high findings. Its core attack: the thesis and the flagship journey depend on DOCX, a format the MVP cannot search or preview, and UJ-1 ends at a step (opening the file) that Find cannot perform. It also argues that the ranking buries the content matches the thesis depends on, that a false "No matching files found" appears while enumeration runs, that the 500 MB budget cannot be measured from a web page, that cloud-synced Documents folders turn indexing into mass downloads, that the index goes stale silently, and that SM-1 cannot validate the thesis. If these reviewers' findings were counted in the grade, it would be Poor, because of the two adversarial criticals.

## Dimension verdicts

- Decision-readiness — strong
- Substance over theater — strong
- Strategic coherence — strong
- Done-ness clarity — adequate
- Scope honesty — strong
- Downstream usability — strong
- Shape fit — strong

## Severity counts by reviewer

| Reviewer | Critical | High | Medium | Low |
|---|---|---|---|---|
| Rubric | 0 | 1 | 5 | 6 |
| EARS | 0 | 3 | 6 | 8 |
| Adversarial | 2 | 8 | 13 | 6 |

## Findings by severity

### Critical (2)

**[Adversarial reviewer]** — C-1 The thesis and the flagship journey depend on a format the MVP cannot search (§1 Vision ¶3; UJ-1; §2.2; FR-16; §7.2; SM-1)
Daniel is looking for a *report* he *worked on*. That is almost certainly a DOCX. In the MVP, DOCX gets no content search (FR-16) and no preview (FR-20), so the climax ("the preview shows the phrase he remembered") cannot happen for the most likely file. Windows Search and Spotlight already index DOCX, XLSX, and PDF content. §2.2 quietly writes off "people whose important content is in Word or Excel files", which describes most of the target user group. Consequence: SM-1 either fails, or "passes" only because the seeded folder avoids Office files, so the MVP proves nothing about real users.
Fix: (a) Move DOCX text extraction into the MVP and add a plain-text DOCX preview; or (b) rewrite UJ-1 around a TXT, MD, or PDF, narrow the persona, and restate the thesis as "faster than OS search for text and PDF content plus name dates". Either way, add a DOCX case to SM-1 or state that it is excluded.

**[Adversarial reviewer]** — C-2 The journey ends in a step Find cannot perform (UJ-1 Resolution; §3 Location; §7.2; addendum Rejected alternatives)
"He notes the file's location … and opens it in his usual app." Find cannot open the file, reveal the full path, or hand the file to the OS. The user must navigate manually in Explorer or Finder from a folder name Find cannot even place on disk: the exact pain the product exists to remove. SM-1 does not say whether "time to the correct file" means identified or opened. Consequence: users hit a dead end at the moment of success, and SM-1 overstates Find's advantage by stopping the clock before the slowest step.
Fix: Define the job's end state. Add the best hand-off the platform allows: Copy relative path, Copy file name, and (if architecture confirms) Download a copy or open-in-new-tab for PDFs and images. Define SM-1's stop condition as "the file is open in its target app", or report both times.

### High (12)

**[Rubric: Done-ness clarity]** — "Edit" is undefined, and FR-10's own headline example fails under the usual definition (FR-10, FR-11, UJ-1 step 2, SM-2)
`reprot` has 6 letters, so it "tolerate[s] 1 edit". Turning `reprot` into `report` is one *transposition*, but under plain Levenshtein distance it costs two substitutions. So FR-10's first consequence, FR-11's typo example, and UJ-1's headline query all fail if the developer chooses the most common algorithm. The PRD also doesn't say whether typo tolerance applies against a token's beginning: does `reprot` match the token `Reports` in `Q3_Reports.pdf`? SM-2, SM-C1 and the OQ-4 tuning depend on both answers.
Fix: In FR-10, define an edit as an insertion, deletion, substitution, or swap of two adjacent letters (Damerau / optimal string alignment distance). State whether a typo-tolerant query word can match a token's beginning, and add one acceptance example either way.

**[EARS reviewer]** — F-01 Token-beginning matching contradicts EARS-01 "names contain the search term" (§3 Name token / Name match; FR-9; FR-8)
EARS-01 describes substring matching; the PRD matches only at token boundaries. `annualreport2025.pdf` is not returned for `report` although its name contains `report`, and `port` does not return `report.pdf`. This silently narrows a "shall display all" requirement and goes against Principle 1.
Fix: (a) Add a substring fallback for query words of 3+ characters, ranked after token-beginning matches, with the test "`report` returns `annualreport2025.pdf`"; or (b) keep token semantics deliberately, amend EARS-01, and record the deviation in a traceability table.

**[EARS reviewer]** — F-02 The query splitting rule is undefined, so `report-final` may not find `report-final.docx` (§3 Query; FR-9)
The Glossary gives detailed splitting rules for names but none for the query. If the query splits only at spaces, `report-final` is one word that matches no single token of `report-final.docx`, breaking EARS-01 on a name from the EARS fixture itself.
Fix: Add to the Glossary: "The query is split into query words by the same rules as name tokens." Add FR-9 tests: "`report-final` matches `report-final.docx`" and "`report.pdf` matches `report.pdf`".

**[EARS reviewer]** — F-03 FR-12 changes EARS-03's message while indexing runs and adds a precondition to its acceptance test (FR-12 bullets 1-2)
EARS-03 is unconditional. The PRD shows "No matches yet. Still searching file contents…" during indexing and adds "after indexing completes" to the test. Because the fixture's `report.pdf` is indexed, the original EARS test is timing-dependent. An undocumented modification of a source requirement.
Fix: (a) Always show "No matching files found" and add the still-updating indicator as a secondary line; or (b) amend EARS-03 with an indexing-complete condition and add a second EARS statement for the indexing state, recorded in a traceability table.

**[Adversarial reviewer]** — H-1 The tokenization rule contradicts its own acceptance test (`q3`) (§3 Name token; FR-9; FR-11)
Names split between letters and digits (`Q3` → `Q`, `3`), so the query word `q3` matches neither token, and at 2 characters it gets no typo tolerance. FR-9 still requires `q3 report` to match `Project_Report_Q3.docx`. A developer who implements the Glossary literally fails FR-9.
Fix: State that query words use the same splitter as names, and that a query word splitting into several tokens matches consecutive name tokens. Add `q3`, `Q3`, and `report2024` fixtures.

**[Adversarial reviewer]** — H-2 "Exact" can be true for a file that is not a name match (§3 Match kind / Name match; FR-11)
`port` against `report.pdf` is a substring hit (so "exact") but no token beginning (so not a name match). Ranking is undefined at the very top of the list; two engineers will build two different rankers, and SM-2 numbers become meaningless.
Fix: Redefine exact as the query's token sequence equalling a consecutive run of name tokens, state that only name matches get a name match kind, and add the `port`/`report.pdf` fixture.

**[Adversarial reviewer]** — H-3 Name search "never waits", but it must wait for folder enumeration, and FR-12 then lies (§1.1 principle 3; FR-1; FR-5; FR-12; FR-15; NFR-4)
Recursive enumeration of tens of thousands of entries takes seconds to minutes (the brief's research records stalls at ~17k files). Still-updating is defined only for content indexing, so a query can show "No matching files found" when the file simply hasn't been listed yet. A false "no match" in the first seconds of every session is the fastest possible way to lose user trust.
Fix: Add an enumeration phase with "Listing files… N found" progress, use still-updating during enumeration or indexing, never show "No matching files found" until both complete or are cancelled, and add enumeration-time targets to NFR-4.

**[Adversarial reviewer]** — H-4 The 500 MB memory budget cannot be measured from a web page (NFR-4 bullet 4; FR-16; §3 Content not searchable)
`performance.memory` is non-standard and imprecise; `measureUserAgentSpecificMemory()` needs cross-origin isolation. The tab can be killed on an 8 GB machine long before 500 MB of index. Untestable as written, and the crashed-tab failure mode is not addressed.
Fix: Restate the budget as a measurable proxy (for example millions of characters of extracted text) enforced by the indexer, add a memory-pressure requirement, and validate the proxy in the architecture spike.

**[Adversarial reviewer]** — H-5 Ranking structurally buries the content matches the thesis depends on (§3 Rank; FR-11; SM-1; SM-3; OQ-3)
Every content-only match ranks below every typo name match, with recency as the only tie-break. `budget memo` can produce dozens of weak typo name matches that outrank the one file that actually contains "budget memo". SM-3 fixtures conveniently avoid competing name matches, and OQ-3 defers this until after the ranking engine is built.
Fix: Define a scored rank (exact/partial name > all-words content match > typo name), add in-tier relevance before recency, add SM-3 fixtures with competing typo matches, and resolve OQ-3 before architecture.

**[Adversarial reviewer]** — H-6 Cloud-synced folders turn "read-only, nothing leaves the machine" into mass downloads (FR-1 MyFiles guide; addendum; NFR-1; NFR-2; §1)
Documents is often redirected to OneDrive Files On-Demand or iCloud Drive. Indexing forces placeholder hydration: gigabytes downloaded, disk filled, network traffic. NFR-1 "passes" while the machine does exactly what the privacy promise implies it won't, and the PRD's recommended folder location walks straight into this.
Fix: Add a requirement for cloud placeholder folders (detect and warn, never index unavailable files or ask first, a "not available offline" FR-16 reason), test redirected folders in the OQ-1 spike, and reconsider Documents as the MyFiles location.

**[Adversarial reviewer]** — H-7 No way to see changes in the folder: the index goes stale silently (§3 Session; FR-3; §4.2; §4.4; principle 4)
Daniel uses Find all day, but files added, renamed, or deleted after enumeration never appear. The only refresh is a reload that throws away the index and permission. By midday the newest files, the ones most likely wanted, are missing with no warning.
Fix: Add a Rescan action that keeps the permission and re-indexes only changed files, show "Listed at HH:MM", and define the preview error for a file that is no longer readable.

**[Adversarial reviewer]** — H-8 SM-1 cannot validate the thesis it is meant to prove (§8 SM-1; §1 Vision ¶3)
With 5 users and a median, no plausible result separates signal from noise. The seeded folder may sit outside the OS search index, handing Find an unfair advantage; content-only tasks are chosen to favor Find; folder choice and indexing time are not stated as included. The MVP's single go/no-go metric produces a result nobody should trust.
Fix: Start the clock in the start state, put the seeded folder in an OS-indexed location, include a DOCX task, report per-task times, and raise n or reframe SM-1 as a directional signal.

### Medium (24)

**[Rubric: Decision-readiness]** — The MyFiles decision rests on an unverified premise and has no success signal (§3 MyFiles folder, FR-1, FR-2 Notes, OQ-1)
The MyFiles folder is "the chosen answer to blocked folders", and OQ-1 says "This no longer blocks UX". But OQ-1's first open item is whether Chrome and Edge can open "a `MyFiles` folder inside Documents on Windows, macOS, and Linux" at all, so the decision's foundation is still unverified. The strategy also asks users to change their browser's download location, which sits uneasily with Principle 7 ("find a file without any onboarding or instructions"), and it only gathers *future* downloads, while UJ-1's Daniel has documents that "pile up across many folders" today. No SM measures whether users complete or benefit from the setup: SM-1 uses a seeded folder.
Fix: State the fallback if the OQ-1 spike fails (for example, the guide recommends another location, or drops the download-location step). Note that the guide is optional and that Principle 7 is satisfied without it. Optionally, record in SM-1 whether participants would adopt MyFiles, next to the OQ-5 re-selection question.

**[Rubric: Strategic coherence]** — Queries that mix a name cue and a content cue match nothing (§3 Name match, Content match; FR-9, FR-14; §2.1)
The core job is "describe what I *do* remember". A name match needs *every* query word in the name, and a content match needs *every* query word in the content. So `budget q3`, where "q3" is in the name and "budget" is only in the text, is neither, and the right file is absent from the result list. Nothing in §6 or §7.2 says this is intended, and it directly affects SM-1's half-remembered-cue tasks and Principle 1 ("Never hide the right file").
Fix: Either decide that a query word may be satisfied by the name *or* the content (and give that case a match kind and a rank), or list "queries that mix name words and content words" in §7.2, and add a mixed-cue query to OQ-3 or the SM-1 task design so the gap is measured.

**[Rubric: Done-ness clarity]** — How a query is split into query words is undefined (§3 Query, FR-9)
The Glossary says the query "is split into **query words**" but not by what. Name tokens split at spaces, `_`, `-`, `.`, case changes, and letter/digit boundaries. If query words split only at spaces, then typing `project_report`, `ProjectReport`, or `Q3` becomes one query word that matches no single name token, so pasting or retyping a real file name can fail.
Fix: State that the query is split by the same rules as name tokens (or name the difference), and add an acceptance example: "`project_report` matches `Project Report.pdf`".

**[Rubric: Done-ness clarity]** — "Exact" uses substring language that the matching rules and EARS-01 don't agree on (§3 Match kind; FR-8 EARS-01; FR-9)
"Exact" is "the whole query appears in the name". EARS-01 requires names that "contain the search term". FR-9 instead matches only whole tokens or token beginnings. For `port` against `report.pdf`, the substring rules say match (and "exact"), while FR-9 says no match. The EARS-01 acceptance fixture passes either way, so the divergence is invisible to its test.
Fix: State that only name matches (FR-9) receive a match kind, and that "exact" means the query appears in the name *and* the entry is a name match. In FR-8, note that the PRD deliberately narrows EARS-01's "contain" to token and token-beginning matching, and add a negative example (`port` does not match `report.pdf`).

**[Rubric: Done-ness clarity]** — The indexing target has no workload definition (NFR-4, SM-5)
"1,000 mixed text files and text-layer PDFs … within 60 seconds" fixes the count but not the sizes or the mix. 1,000 PDFs near the 10 MB limit and 1,000 small `.md` files differ by orders of magnitude, so the target can't be tested and can't guide architecture.
Fix: Define the SM-5 fixture: share of PDFs, median and maximum file size, and total bytes (for example "≤ 500 MB total, 30% PDFs, median 200 KB").

**[EARS reviewer]** — F-04 Message precedence is undefined when filters hide every match (FR-12 bullet 3)
Unspecified whether "No matching files found" is suppressed when matches exist but are all filtered out, or whether "Filters hide 0 matches" appears.
Fix: "No matching files found" only when the query matches zero entries before filters; when all matches are filtered out, show only "Filters hide N matches" with Clear filters; never show "Filters hide 0 matches".

**[EARS reviewer]** — F-05 The empty-result message after indexing is cancelled is undefined (FR-12; FR-13 bullet 3)
After a cancel, indexing is neither "complete" nor "running", so neither FR-12 message applies.
Fix: Add: after cancel, an empty result list for a non-empty query shows "No matching files found" with the content-search-incomplete notice.

**[EARS reviewer]** — F-06 EARS-02's "preview panel" is not reconciled with the Glossary term "preview" (§3 Preview; FR-20)
The Glossary's preview exists only with a selected file entry, so "the preview shows …" with none selected is self-contradictory. UX has no defined region name for the EARS-02 message.
Fix: Add a Glossary entry for **Preview panel** and reword the FR-20 bullet to "the preview panel shows …".

**[EARS reviewer]** — F-07 EARS-02's state is underspecified, and the PRD's narrower condition is not stated as a deviation (FR-20; §3 Start state)
EARS-02 applies whenever no file is selected, including the start state, where the PRD shows no preview. The PRD's "with a folder selected" is sensible but a silent narrowing.
Fix: Restate EARS-02 in the PRD with the folder-open condition and note the narrowing in the traceability table.

**[EARS reviewer]** — F-08 The PRD does not restate the EARS statements or log its deviations (§0)
Only the acceptance tests are reproduced. The PRD changes semantics (F-01, F-03, F-07) without saying so.
Fix: Add an "EARS Traceability" subsection: original statement, refined EARS statement, carrying FR, deviations and rationale.

**[EARS reviewer]** — F-09 NFR-3's storage list is not exhaustive (NFR-3)
Omits sessionStorage (survives reload, undermining FR-3) and the Origin Private File System (a likely spill target for a 500 MB index).
Fix: Reword to "any browser storage, including but not limited to …" with session storage, OPFS and service-worker caches; extend Verify.

**[Adversarial reviewer]** — M-1 The indexing progress total is unknown while enumeration runs (FR-13)
The denominator grows while the user watches; the progress bar runs backward or jumps.
Fix: Show "340 indexed, 1,200+ found (still listing)" until enumeration completes.

**[Adversarial reviewer]** — M-2 Behavior at the 50,000-entry limit is undefined, and it hides files (NFR-4 bullet 2; principle 1)
Files beyond the limit vanish with no explanation; each story writer will invent their own behavior.
Fix: Stop at the limit, keep what is listed, show a persistent banner, and treat the list as incomplete in FR-12 messages.

**[Adversarial reviewer]** — M-3 A query split across the name and the content matches nothing (§3 Name match / Content match; FR-9; FR-14)
For `budget daniel`, with `budget` in the name and `daniel` only inside the file, the file is neither kind of match. The right file is hidden for exactly the half-remembered queries the product targets.
Fix: Add a combined name + content match kind and place it in the rank.

**[Adversarial reviewer]** — M-4 The text preview cut-off can sit before the first match (FR-20; FR-16)
A 5 MB log is indexed in full but previewed only to 1 MB; a match at 3 MB cannot be shown. Users conclude the match is wrong.
Fix: Preview a 1 MB window around the first match and say so in the cut-off notice.

**[Adversarial reviewer]** — M-5 PDF preview safety is not specified enough to meet NFR-1 and NFR-5 (NFR-5; NFR-1; FR-20; addendum)
pdf.js supports PDF JavaScript and link annotations; one click in a hostile PDF breaks the privacy guarantee.
Fix: Require scripting disabled, inert or confirmed links, no form submission; add to SM-6.

**[Adversarial reviewer]** — M-6 pdf.js support files make "zero requests after load" fragile (NFR-1; SM-6; addendum)
Worker, CMaps, standard fonts and lazy chunks load on demand; SM-6 fails on the first CJK or non-embedded-font PDF.
Fix: Fetch or inline every runtime asset at initial load; add CJK and non-embedded-font PDFs to SM-6.

**[Adversarial reviewer]** — M-7 Cancelling the picker mid-session may destroy the session (FR-2; FR-3; §3 Session)
Read literally, one stray click on Choose folder then Cancel throws away a minutes-long index.
Fix: A cancelled or blocked picker during an active session keeps the session and shows the hint inline.

**[Adversarial reviewer]** — M-8 Name-date detection will produce false positives that SM-C3 says it must not (FR-18; SM-C3; addendum)
`invoice_2048.pdf`, `PO-2075.pdf`, `v2024` all become dates; digit boundaries are undefined.
Fix: Define boundaries precisely and add these cases to SM-7 and SM-C3 fixtures.

**[Adversarial reviewer]** — M-9 Developer and junk folders are not excluded (§3 Type; NFR-4; FR-13)
`node_modules`, `.git`, build output and `~$` lock files blow the 50k limit and memory budget and pollute results.
Fix: Add a visible default exclusion list with an excluded-entry count.

**[Adversarial reviewer]** — M-10 Text encodings are unstated, so content search fails silently (FR-14; FR-16; §3 Searchable content)
UTF-16 and Windows-1252 CSVs decoded as UTF-8 never match, yet are not labelled, which SM-C2 forbids.
Fix: Require BOM and encoding detection and add "could not decode text" as an FR-16 reason.

**[Adversarial reviewer]** — M-11 The "content not searchable" label covers DOCX and XLSX only (FR-16; §3 Content not searchable)
PPTX, RTF, ODT, EML, MSG carry no label; users trust a negative result for a deck OS search would have found.
Fix: Label every non-image type other, or list the labelled formats.

**[Adversarial reviewer]** — M-12 The layout relationship between the folder tree and the result list is contradictory (§4.2; FR-12; FR-19; FR-7)
FR-12 implies a swap; §4.2 and FR-19 imply both visible. UX and stories will argue about the layout.
Fix: State the layout model or mark it as a UX decision with constraints.

**[Adversarial reviewer]** — M-13 Accessibility for a large virtualized tree has no testable criteria (NFR-6; FR-5; FR-15)
Virtualization, ARIA tree patterns and live-region announcements are unstated; accessibility gets marked done by assertion.
Fix: Add concrete criteria: ARIA tree pattern, throttled polite live region, focus retention during inserts, a named screen reader/browser pair.

### Low (20)

**[Rubric: Done-ness clarity]** — Behavior above 50,000 file entries is undefined (NFR-4)
"Folders larger than that are not supported in the MVP, and Find says so" doesn't say whether Find stops reading, loads the first 50,000, or refuses the folder, or whether name search and indexing run.
Fix: One consequence, for example "Find stops at 50,000 file entries, shows a notice suggesting a subfolder, and searches only what was loaded".

**[Rubric: Done-ness clarity]** — Name-date edge cases are open (FR-18, SM-7, SM-C3)
"Standalone year" isn't tied to name tokens, which `ticket_20251` (SM-C3) depends on. Names with two dates have no rule. The addendum's valid-calendar-date rule is a product rule that SM-7 tests, but it lives only in the addendum.
Fix: Define "standalone" as "a name token of exactly four digits", say which date wins when there are several (or that any date counts), and move the valid-calendar-date rule into FR-18.

**[Rubric: Done-ness clarity]** — The accessibility scope for previews is unclear (NFR-6, FR-20)
"Fully usable by keyboard, and meet WCAG 2.1 AA" for "the preview" covers PDFs rendered page by page and arbitrary user images with no alt text. Read literally, this needs an accessible PDF text layer, a significant architecture input.
Fix: Scope it: preview controls and text-file previews meet WCAG 2.1 AA; PDF and image previews are keyboard-operable, with the file name as the accessible label.

**[Rubric: Done-ness clarity]** — "Filters hide N matches" is ambiguous with an empty query (FR-12)
With no query there are no matches, yet the filtered-out notice "applies whether the visible result list is empty or not".
Fix: Say that with an empty query the count is of file entries hidden by the filters, or that the notice only appears when there is a query.

**[Rubric: Scope honesty]** — §0 promises a "Blocks" marker that no OQ carries (§0 How to read it; §9)
§0 says "Open questions that block a later phase are marked **Blocks** in §9", but §9 uses only "Informs", and OQ-1 explicitly "no longer blocks UX".
Fix: Say in §0 that no open question currently blocks a phase, or mark OQ-1 as blocking the architecture spike if that is the intent.

**[Rubric: Downstream usability]** — Residual synonyms of Glossary terms (§1, §2.2, UJ-1 step 4, §4.5, FR-1, FR-4, NFR-3, NFR-4, OQ-5)
"Results" / "name results" stands in for the result list or file entries in it. "Folder access" and "folder permission" sit next to the defined **read-only access**. Story extraction will copy them.
Fix: Replace "results" with "the result list" where it refers to the list, and use "read-only access" (or add "folder permission" to the Glossary as the persisted browser handle OQ-5 is about).

**[EARS reviewer]** — F-10 The EARS-01 acceptance wording is weaker than "Only" (FR-8 bullet 1)
Equivalent for a three-file fixture, but does not carry "Only" if the fixture grows.
Fix: Write "returns exactly `report.pdf` and `report-final.docx`".

**[EARS reviewer]** — F-11 NFR-1 scopes "no network" to after the initial load (NFR-1)
Correct reading, but undocumented against the EARS constraint.
Fix: Add one line explaining the reading: only Find's own static assets, never a request carrying file data.

**[EARS reviewer]** — F-12 The fixture file content is unspecified (FR-8, FR-12)
If the fixture `report.pdf` contained `xyz123`, EARS-03 would fail with a content match.
Fix: Specify fixtures: `report.pdf` is a text-layer PDF that does not contain `xyz123`.

**[EARS reviewer]** — F-13 EARS-01 "all files" vs the NFR-4 capacity cap (NFR-4 bullet 2)
Above 50,000 file entries the PRD does not satisfy "all files".
Fix: Note the bound in the traceability table; specify what "not supported" means.

**[EARS reviewer]** — F-14 EARS form quality: EARS-01 trigger and statement/test mismatch (docs/product/earsRequirments EARS-01)
"Enters" is ambiguous; the statement requires "all" while the test asserts "only"; "contain" fixes substring semantics.
Fix: Restate as "When the user changes the query, …" stating completeness and allowed extras.

**[EARS reviewer]** — F-15 EARS form quality: EARS-03 pattern choice and ambiguity (EARS-03; FR-12)
No-match is a normal outcome, not an unwanted event; "match" and "non-empty" are ambiguous; "search results area", "search field", "search box" are not Glossary terms.
Fix: Restate in event-driven form against the result list; define a blank query; add "search box" and "result list" to the Glossary.

**[EARS reviewer]** — F-16 Deselection rules affecting EARS-02 are unspecified (FR-15, FR-12, FR-20)
Unclear whether the preview panel returns to "Select a file to preview" when the selected entry is filtered out, the query changes, or a folder is selected.
Fix: Add an FR-20 consequence: the selected file entry stays selected until another is selected or the session ends.

**[EARS reviewer]** — F-17 Project name: "Smart Local File Browser" vs "Find" (§0; EARS file header)
A reader matching documents by project name would not connect them.
Fix: Add a clause to §0 noting the EARS file's project name.

**[Adversarial reviewer]** — L-1 Other Chromium browsers fall between FR-4 and NFR-7 (FR-4; NFR-7; addendum)
Opera, Vivaldi and Arc pass feature detection; Brave disables the API.
Fix: Base FR-4 on API availability and reword the message.

**[Adversarial reviewer]** — L-2 The user's sort and live inserts conflict (FR-7; FR-15)
New content matches are inserted in rank order even when the user has sorted.
Fix: Insert in the active order.

**[Adversarial reviewer]** — L-3 The 90% coverage target has no defined boundary (NFR-8; SM-8)
"Rules" and the coverage measure are undefined; easy to game.
Fix: Name the rule modules and specify branch coverage.

**[Adversarial reviewer]** — L-4 Time zone and day boundaries for the date filter are unstated (FR-18; FR-6)
Off-by-one-day filter results and flaky tests.
Fix: Evaluate ranges in local time, 00:00:00 start to 23:59:59.999 end.

**[Adversarial reviewer]** — L-5 "Final" status with unverified foundations (front matter; OQ-1; §10)
The MyFiles strategy and the "New folder in picker" claim are unverified; FR-1 and FR-2 change if the spike fails.
Fix: Mark FR-1 and FR-2 conditional on the OQ-1 spike and name a fallback.

**[Adversarial reviewer]** — L-6 The assumptions index is incomplete (§10; FR-12; FR-6; FR-13)
Exact message strings, size-format thresholds and filter-notice behavior are untagged; downstream treats placeholder copy as requirements.
Fix: Tag them, or state that UI copy is a UX decision.

## Mechanical notes

- **Assumptions Index roundtrip: clean.** Every inline `[ASSUMPTION]` has an index entry, and every index entry appears inline. The index order is not by section (FR-1 sits after FR-18). Cosmetic.
- **IDs.** FR-1 to FR-20, NFR-1 to NFR-8, SM-1 to SM-8, SM-C1 to SM-C3, OQ-1 to OQ-8, and UJ-1 are contiguous and unique. All cross-references resolve.
- **Resolved OQs kept in §9.** OQ-6 and OQ-8 are marked *Resolved*. Moving them to a short "Resolved" sub-list would keep §9 to open questions only.
- **Provenance note inside a requirement.** FR-19 says "Confirmed by the user." This is process history, not a requirement. Drop it, or move it to the memlog.
- **Brief sync verified.** The brief matches the PRD on MyFiles, the smart date filter, and version clusters (post-MVP), so OQ-8's resolution holds.
- **Required sections** are all present for the stakes and product type.
- **Pre-final review status.** All findings in `review-rubric-prefinal.md` are resolved in the current text. Only partial carry-overs remain: a few "results" and "folder access" synonyms (Downstream usability, low), and the phase-blocker marker that §0 promises but §9 doesn't use (Scope honesty, low).

## Reviewer files

- `review-rubric.md`
- `review-ears.md`
- `review-adversarial-general.md`
