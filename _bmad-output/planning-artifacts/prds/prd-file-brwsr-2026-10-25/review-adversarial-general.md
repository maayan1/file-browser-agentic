---
title: "Adversarial Review: PRD Find"
reviewed: prd.md, addendum.md
lens: adversarial (bmad-review)
date: 2026-10-25
---

# Adversarial Review: PRD Find

Content class: docs (behavior-defining PRD). Lens: adversarial. The bmad-review lens is unranked by default; the caller asked for severity, so each finding carries one here. Every finding has a location, the problem, a fix, and the consequence if it ships unchanged.

**Counts:** 2 critical, 8 high, 13 medium, 6 low (29 total).

**The core attack:** the PRD wants to prove that Find "gets users to the right file faster than their operating system's search". Yet it excludes the formats that office workers' important files mostly use (DOCX, XLSX). It ends the flagship journey at a step Find cannot perform. It ranks content matches below every weak name match. And it compares itself with OS search tools that already index content. The thesis is set up to fail, or to pass only on a rigged fixture.

---

## Critical

### C-1. The thesis and the flagship journey depend on a format the MVP cannot search
- **Location:** §1 Vision ¶3; §2.3 UJ-1 (persona, step 2, climax); §2.2 Non-Users bullet 3; FR-16; §7.2 bullet 3; SM-1.
- **Problem:** Daniel is looking for a *report* he *worked on*. That is almost certainly a DOCX (or a PPTX or XLSX). In the MVP, DOCX is type **other**: it gets no content search (FR-16) and no preview ("Preview not available", FR-20). So the climax ("the preview shows the phrase he remembered") cannot happen for the most likely file. Meanwhile, Windows Search and Spotlight already index DOCX, XLSX, and PDF content. §2.2 quietly writes off "people whose important content is in Word or Excel files", which describes most of the target user group in §1.
- **Fix:** Pick one. (a) Move DOCX text extraction into the MVP (a ZIP plus XML text pull is small and local) and add a DOCX preview as plain extracted text. Or (b) rewrite UJ-1 so the target is a TXT, MD, or PDF, narrow the persona to people whose files really are text or PDF, and restate the thesis as "faster than OS search for text and PDF content plus name dates". Whichever you pick, add a DOCX case to SM-1 or state explicitly that it is excluded.
- **Consequence:** SM-1 either fails, or "passes" only because the seeded folder avoids Office files, so the MVP proves nothing about real users. UX designs a climax that real users never see.

### C-2. The journey ends in a step Find cannot perform
- **Location:** §2.3 UJ-1 Resolution; Glossary "Location"; §7.2 "Full on-disk paths"; addendum, Rejected alternatives ("Opening files in their native apps").
- **Problem:** The resolution says "he notes the file's location (shown from the selected folder's name down) and opens it in his usual app." Find cannot open the file, cannot reveal the full path, and cannot hand the file to the OS. The user has to switch to Explorer or Finder and navigate manually from a folder name Find cannot even place on disk. That manual navigation is the exact pain the product exists to remove. SM-1 measures "time to the correct file" without saying whether that means *identified in Find* or *opened*.
- **Fix:** Define the job's end state explicitly. Add a requirement for the best hand-off the platform allows: a **Copy relative path** action, a **Copy file name** action (paste into the OS search or the "Open" dialog of the target app), and, if the architecture confirms it works, a **Download a copy** or open-in-new-tab action for PDFs and images that the browser can display. Define SM-1's stop condition as "the file is open in its target app", or report both times.
- **Consequence:** Users hit a dead end at the moment of success. The SM-1 result overstates Find's advantage because it stops the clock before the slowest step.

---

## High

### H-1. The tokenization rule contradicts its own acceptance test (`q3`)
- **Location:** Glossary "Name token"; FR-9 consequence 2; FR-11 example.
- **Problem:** Names are split "between letters and digits (`Q3` → `Q`, `3`)". The query word `q3` therefore matches neither token `q` nor token `3`, exactly or as a beginning. It is 2 characters long, so it gets no typo tolerance either. FR-9 still requires `q3 report` to match `Project_Report_Q3.docx`. The Glossary never says whether the *query* is tokenized with the same rules.
- **Fix:** State that query words are tokenized with the same splitter as names, and that a query word that splits into several tokens matches when those tokens match consecutive name tokens. Add `q3`, `Q3`, and `report2024` as fixtures.
- **Consequence:** A developer who implements the glossary literally fails FR-9. Story writers get two contradictory sources of truth.

### H-2. "Exact" can be true for a file that is not a name match
- **Location:** Glossary "Match kind: exact" vs "Name match"; FR-11.
- **Problem:** "exact" means "the whole query appears in the name, ignoring case", which is a substring test. "Name match" needs every query word to match a token exactly or as its beginning. The query `port` against `report.pdf` is a substring hit (so it would be "exact"), but it matches no token beginning (so it is not a name match). Is it in the result list? Ranked first? Absent? Also, `report` against `report-final.docx` and against `myreport.pdf` would both be "exact" under the substring rule.
- **Fix:** Redefine **exact** as "the query's token sequence equals a consecutive run of name tokens" (or "equals the whole base name"), and state that only name matches get a name match kind. Add the `port`/`report.pdf` fixture with its expected result.
- **Consequence:** Ranking is undefined at the very top of the list. Two engineers will build two different rankers, and SM-2 numbers become meaningless.

### H-3. Name search "never waits", but it must wait for folder enumeration, and FR-12 then lies
- **Location:** §1.1 principle 3; UJ-1 step 1; FR-1 consequence 3; FR-5; FR-12 ("No matching files found"); FR-15; NFR-4; addendum "Research" (~17k files stalling).
- **Problem:** Name search can only search entries that have already been enumerated. With the File System Access API, recursive enumeration of tens of thousands of entries takes seconds to minutes, and the brief's own research records stalls at ~17k files. The PRD defines *still updating* only for content indexing. While enumeration runs, a query can show "No matching files found" when the file simply hasn't been listed yet. That breaks principle 1 ("never hide the right file") and FR-12.
- **Fix:** Add an explicit **enumeration** phase to the Glossary and to FR-13 and FR-15: show "Listing files… N found" progress, and use *still updating* while either enumeration or indexing runs. Never show "No matching files found" until both are complete or cancelled. Add a target for full enumeration time at 10k and 50k entries to NFR-4.
- **Consequence:** A false "no match" in the first seconds of every session, on exactly the large folders that motivated the product. That is the fastest possible way to lose user trust.

### H-4. The 500 MB memory budget cannot be measured from a web page
- **Location:** NFR-4 bullet 4; FR-16 reason "memory budget reached"; Glossary "Content not searchable".
- **Problem:** A page cannot reliably read its own heap usage. `performance.memory` is non-standard and imprecise, and `measureUserAgentSpecificMemory()` needs cross-origin isolation and returns results asynchronously and coarsely. A worker-held index plus 50k file entries plus pdf.js page renders can also get the tab killed (the "Aw, Snap!" page) on an 8 GB machine long before 500 MB of *index* is reached. The requirement is untestable as written, and the failure mode, a crashed tab that loses the whole session, is not addressed.
- **Fix:** Restate the budget as a measurable proxy, such as "at most N million characters of extracted text" (or a byte count of stored strings), enforced by the indexer. Add a requirement for what happens when the tab is under memory pressure. Have the architecture spike validate the proxy against real heap snapshots.
- **Consequence:** Architecture cannot implement the label honestly. QA cannot verify NFR-4. Users can lose all session state to an out-of-memory crash, with no explanation.

### H-5. Ranking structurally buries the content matches the thesis depends on
- **Location:** Glossary "Rank" [ASSUMPTION]; FR-11; SM-1 ("at least 2 of 5 tasks solvable only through content"); SM-3; OQ-3.
- **Problem:** Every content-only match ranks below every typo name match, and ties within a tier go to recency, with no relevance signal at all. With typo tolerance of 1–2 edits on short words, a query like `budget memo` can produce dozens of weak typo or partial name matches that all outrank the one file that actually contains "budget memo". SM-3 ("top 5 at least 90%") is measured on fixtures where the words appear *only* inside the target, which conveniently avoids competing name matches. OQ-3 defers this to the SM-1 test, which is far too late to change a ranking model.
- **Fix:** Define a scored rank, not strict tiers. At minimum, rank an exact or partial name match > a content match whose words *all* appear > a typo name match, and add an in-tier relevance score (token coverage, edit distance, term frequency) before recency. Add SM-3 fixtures where competing typo name matches exist. Resolve OQ-3 before architecture, not after the user test.
- **Consequence:** Content search, the product's differentiator, is invisible below the fold on real folders. SM-1's content tasks fail. The fix arrives after the ranking engine is built.

### H-6. Cloud-synced folders turn "read-only, nothing leaves the machine" into mass downloads
- **Location:** FR-1 MyFiles guide (Documents); addendum "MyFiles setup" (`startIn: "documents"`); NFR-1; NFR-2; §1 Vision ("safe to use on any work computer").
- **Problem:** On many work Windows machines, Documents is redirected to OneDrive with Files On-Demand. On macOS, iCloud Drive "Desktop & Documents" works the same way. Reading every text file and PDF for indexing forces the OS to *hydrate* cloud-only placeholders, which downloads gigabytes, fills the disk, and generates network traffic. Find's own requests stay at zero, so NFR-1 "passes" while the machine does exactly what the privacy promise implies it won't. Reads can also fail or hang on placeholders. The PRD's recommended folder location walks straight into this.
- **Fix:** Add an NFR or FR that covers cloud placeholder folders: detect and warn where possible, never index files that are not available locally (or ask first), and state this in FR-16 as a reason ("not available offline"). Have the OQ-1 spike test OneDrive and iCloud redirected Documents folders. Reconsider recommending Documents as the MyFiles location.
- **Consequence:** A user points Find at Documents and sees disk usage spike and a corporate network flagged. The trust job (§2.1) collapses, and the "safe on any work computer" claim is false.

### H-7. No way to see changes in the folder: the index goes stale silently
- **Location:** Glossary "Session"; FR-3; §4.2; §4.4; §1.1 principle 4 (Transparent results).
- **Problem:** Session is defined as lasting until reload, close, or a new folder, and Daniel is expected to use Find "throughout the workday". Files added, renamed, or deleted after enumeration never appear. The only refresh is a reload, which throws away the content index and the folder permission. Previewing a deleted file fails, and the PRD defines no error for that case. Nothing tells the user that results reflect the folder as it was at some earlier time.
- **Fix:** Add an FR for a **Rescan** action that keeps the permission, re-enumerates, and re-indexes only changed files (by modified date and size). Show "Listed at HH:MM". Define the preview error when a file is no longer readable. (The `FileSystemObserver` API can be noted in the addendum as optional.)
- **Consequence:** By midday the newest files, the ones the user most likely wants, are missing from results with no warning. That breaks principles 1 and 4.

### H-8. SM-1 cannot validate the thesis it is meant to prove
- **Location:** §8 SM-1; §1 Vision ¶3.
- **Problem:** With 5 users and a median, no plausible result separates signal from noise. The seeded folder is unfamiliar to every participant, and it may sit outside the OS search index (Windows Search indexes only some locations by default), which hands Find an unfair advantage. The required "tasks solvable only through content or a name date" are chosen to favor Find. The time to choose the folder and wait for indexing is not stated as included, yet it is unavoidable for Find on every session, while OS search has none.
- **Fix:** Specify the following. The clock starts with Find in the start state (folder choice and indexing included). The seeded folder must be in an OS-indexed location, with indexing confirmed complete before the test. Include at least one DOCX task (see C-1). Report per-task times. Either raise n or reframe SM-1 as a directional qualitative signal, not proof.
- **Consequence:** The MVP's single go/no-go metric produces a result nobody should trust. Product decisions (OQ-3, OQ-5, version clusters) are then made on noise.

---

## Medium

### M-1. The indexing progress total is unknown while enumeration runs
- **Location:** FR-13 consequence 2 ("Indexing 340 / 1,200 files").
- **Problem:** The total number of text files and PDFs is unknown until enumeration finishes, so the denominator grows while the user watches.
- **Fix:** Show "340 indexed, 1,200+ found (still listing)" until enumeration completes. Specify that behavior.
- **Consequence:** The progress bar runs backward or jumps. Users read it as a bug.

### M-2. Behavior at the 50,000-entry limit is undefined, and it hides files
- **Location:** NFR-4 bullet 2; §1.1 principle 1.
- **Problem:** "Folders larger than that are not supported, and Find says so." Find only learns this after enumerating 50k entries. Does it stop listing (so it silently drops files beyond the limit), refuse the folder, or carry on in a degraded mode?
- **Fix:** State the behavior: stop at the limit, keep what is listed, show a persistent "Only the first 50,000 files are searched. Choose a subfolder." banner, and treat the list as incomplete in FR-12 messages.
- **Consequence:** Files beyond the limit vanish with no explanation, which breaks principle 1. Each story writer will invent their own behavior.

### M-3. A query split across the name and the content matches nothing
- **Location:** Glossary "Name match" and "Content match"; FR-9; FR-14.
- **Problem:** Both kinds of match need *every* query word on one side. For `budget daniel`, where `budget` is in the name and `daniel` is only inside the file, the file is neither a name match nor a content match. Users describe what they remember, which is a mix of both (§2.1 core job).
- **Fix:** Add a combined match: every query word matches either a name token or the content. Label it (for example **name + content**) and place it in the rank.
- **Consequence:** The right file is hidden for exactly the half-remembered queries the product targets.

### M-4. The text preview cut-off can sit before the first match
- **Location:** FR-20 consequence 1 (1 MB cut-off) vs consequence 5 (scroll to first matched words); FR-16 (10 MB index limit).
- **Problem:** A 5 MB log is indexed in full, but the preview stops at 1 MB. If the first match is at 3 MB, the preview cannot scroll to it, and the climax of UJ-1 fails.
- **Fix:** Preview a 1 MB window *around the first match* instead of the first 1 MB, and say so in the cut-off notice.
- **Consequence:** A "content" match with a preview that does not show the content. Users conclude the match is wrong.

### M-5. PDF preview safety is not specified enough to meet NFR-1 and NFR-5
- **Location:** NFR-5; NFR-1; FR-20 (PDFs); addendum "Safe previews".
- **Problem:** pdf.js supports JavaScript in PDFs (`enableScripting`) and renders link annotations. One click on a link inside a PDF navigates or opens a network request, which violates NFR-1 "zero network requests". The addendum covers text and SVG only.
- **Fix:** Require PDF scripting to be disabled, link annotations to be inert or confirmed before opening in a new tab (and declare that out of NFR-1 scope), and no form submission. Add these to the SM-6 automated check.
- **Consequence:** A privacy guarantee that one click in a hostile or tracking PDF breaks.

### M-6. pdf.js support files make "zero requests after load" fragile
- **Location:** NFR-1; SM-6; addendum "PDF".
- **Problem:** pdf.js normally loads its worker, CMaps, and standard font data on demand, usually the first time a PDF is opened, not at page load. Lazy-loaded app chunks behave the same way. SM-6 then fails on the first CJK or non-embedded-font PDF.
- **Fix:** State that every runtime asset (worker, CMaps, standard fonts, code chunks) is fetched or inlined at initial load, and add a CJK PDF and a non-embedded-font PDF to the SM-6 session script.
- **Consequence:** NFR-1 fails in QA late, or someone weakens it quietly to "no requests except our own assets".

### M-7. Cancelling the picker mid-session may destroy the session
- **Location:** FR-2 ("returns to the start state"); FR-3; Glossary "Session".
- **Problem:** If a user with a fully indexed folder clicks **Choose folder** and then cancels, FR-2 says Find returns to the start state. Read literally, that throws away a minutes-long index because of one stray click.
- **Fix:** State that a cancelled or blocked picker during an active session keeps the current session, and shows the hint inline.
- **Consequence:** Data loss from one mis-click, and a story that implements the requirement literally.

### M-8. Name-date detection will produce false positives that SM-C3 says it must not
- **Location:** FR-18 recognized formats; SM-C3; addendum "Name-date detection".
- **Problem:** "A standalone year from 1990 to 2099" turns `invoice_2048.pdf`, `room_2010.docx`, `PO-2075.pdf`, and `1920x1080_2000.png` into dates. "Standalone" and the digit boundaries for `YYYYMMDD` (for example `ref_202512319.pdf`) are undefined, and the tokenizer splits letters from digits, so `v2024` becomes year 2024.
- **Fix:** Define boundaries precisely (separator or name start or end on both sides, and no adjacent digits). Decide whether a letter-digit boundary counts. Add the cases above to the SM-7 and SM-C3 fixtures.
- **Consequence:** The default date source **Modified or date in name** lets wrong files through the filter, undermining the "smart" date filter.

### M-9. Developer and junk folders are not excluded
- **Location:** Glossary "Type: text file" (code extensions); NFR-4; FR-13.
- **Problem:** Code extensions count as text files, and nothing excludes `node_modules`, `.git`, build output, or Office lock files (`~$report.docx`). A single project folder can exceed 50k entries and fill the index with noise, and lock files show up as top "exact" matches.
- **Fix:** Add a default exclusion list (dot-folders, `node_modules`, `~$*`, `Thumbs.db`, `.DS_Store`), show it, and let the user see how many entries it excluded (principle 1).
- **Consequence:** Slow indexing, a blown memory budget, and polluted results for anyone with code in the selected folder.

### M-10. Text encodings are unstated, so content search fails silently
- **Location:** FR-14; FR-16; Glossary "Searchable content".
- **Problem:** CSV files from Excel are often UTF-16 or Windows-1252, and logs vary too. Decoding everything as UTF-8 produces garbage that never matches, yet the file is not labelled **content not searchable**.
- **Fix:** Require BOM and encoding detection (UTF-8, UTF-16 LE and BE, and a Windows-1252 fallback), and add "could not decode text" as a FR-16 reason.
- **Consequence:** Silently unsearched files, which SM-C2 explicitly forbids.

### M-11. The "content not searchable" label covers DOCX and XLSX only
- **Location:** FR-16 consequence 2; Glossary "Content not searchable" ("format not supported yet").
- **Problem:** PPTX, RTF, ODT, EML, MSG, and similar files are type **other** with no label, so users assume their content was searched. That is the exact misunderstanding FR-16 exists to prevent for Word files.
- **Fix:** Label every non-image file of type **other** "format not supported yet", or define and list the formats that get the label.
- **Consequence:** Users trust a negative result for a PowerPoint deck they would have found with OS search.

### M-12. The layout relationship between the folder tree and the result list is contradictory
- **Location:** §4.2 description ("browse it at any time, with or without a query"); FR-12 ("With an empty query and no active filter, Find shows the folder tree, not a result list"); FR-19; FR-7.
- **Problem:** Is the folder tree a permanent panel, or does it swap places with the result list? FR-12 implies the swap. §4.2 and FR-19 imply both are visible at once. The "selected file entry" can come from either view, and the PRD does not say what happens to it when the view changes.
- **Fix:** State the layout model in the PRD (for example: the folder tree is always visible on the left, the result list replaces the main pane, the preview is on the right), or mark it explicitly as a UX decision with constraints.
- **Consequence:** UX and stories argue about the layout. FR-12 and FR-19 tests are written against different layouts.

### M-13. Accessibility for a large virtualized tree has no testable criteria
- **Location:** NFR-6; FR-5; FR-15.
- **Problem:** "Fully usable by keyboard and WCAG 2.1 AA" for a 10k–50k node tree and a live-updating result list depends on virtualization, ARIA tree patterns, and live-region announcements. FR-15's "insert without changing scroll position" adds focus and announcement problems. None of this is stated.
- **Fix:** Add concrete criteria: the ARIA tree pattern with arrow-key navigation, a polite live-region announcement of result counts (throttled), focus kept on the selected file entry during inserts, and a named screen reader and browser pair for testing.
- **Consequence:** Accessibility gets marked "done" by assertion, or it fails late after virtualization is built.

---

## Low

### L-1. Other Chromium browsers fall between FR-4 and NFR-7
- **Location:** FR-4; NFR-7; addendum "Feature-detect".
- **Problem:** Feature detection lets Opera, Vivaldi, and Arc through, while Brave disables the API by default. The message says Chrome or Edge is "required", but the behavior depends on the API being present, not on the browser.
- **Fix:** Base FR-4 on API availability, word the message as "a browser that supports folder access, such as Chrome or Edge", and keep NFR-7 as the *tested* matrix.
- **Consequence:** Confusing messages and support questions from Brave users.

### L-2. The user's sort and live inserts conflict
- **Location:** FR-7 consequence 2; FR-15 consequence 1.
- **Problem:** New content matches are "inserted in rank order", but the user may have replaced rank order with a sort.
- **Fix:** Insert in the active order, whichever it is.
- **Consequence:** New entries appear out of order in a sorted list.

### L-3. The 90% coverage target has no defined boundary
- **Location:** NFR-8; SM-8.
- **Problem:** "Rules" and the coverage measure (line or branch) are not defined, so the number can be met by drawing the boundary narrowly.
- **Fix:** Name the rule modules (tokenizer, matcher, ranker, filters, name-date detector, index manager) and specify branch coverage.
- **Consequence:** A portfolio metric that is easy to game.

### L-4. Time zone and day boundaries for the date filter are unstated
- **Location:** FR-18; FR-6.
- **Problem:** Inclusive ranges against modified timestamps need a time zone. Files modified at 23:30 UTC can fall on the wrong day.
- **Fix:** State that ranges are evaluated in the browser's local time zone, from 00:00:00 on the start day to 23:59:59.999 on the end day.
- **Consequence:** Off-by-one-day filter results and flaky tests.

### L-5. "Final" status with unverified foundations
- **Location:** front matter `status: final`; OQ-1 ("no longer blocks UX"); §10 (15 assumptions).
- **Problem:** The MyFiles strategy, the answer to blocked folders, is unverified on all three operating systems, and the addendum's "New folder in picker" claim is also unverified. If the spike fails, FR-1 and FR-2 change.
- **Fix:** Mark FR-1 and FR-2 as conditional on the OQ-1 spike, and name a fallback (for example, guide the user to any subfolder of Documents).
- **Consequence:** UX builds onboarding around a setup that may not work on macOS or Linux.

### L-6. The assumptions index is incomplete
- **Location:** §10; FR-12 message texts; FR-6 size format; FR-13 cancel behavior.
- **Problem:** Several invented specifics (exact message strings, the size-format thresholds, the "Filters hide N matches" behavior) are not tagged, while weaker ones are.
- **Fix:** Tag them, or state that UI copy is a UX decision.
- **Consequence:** Downstream workflows treat placeholder copy as a requirement.
