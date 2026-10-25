---
title: "PRD: Find"
status: final
created: 2026-10-25
updated: 2026-10-25
---

# PRD: Find

> **Core principle:** Users should not need to remember a file's exact name or location to find it.

## 0. Document Purpose

This PRD is for whoever designs, architects, and builds Find, including the downstream BMAD workflows: UX, architecture, and epics and stories.

It builds on these inputs and does not repeat them:
- The product brief: `_bmad-output/planning-artifacts/briefs/brief-file-brwsr-2026-10-25/`.
- The EARS requirements: `docs/product/earsRequirments`. EARS-01 to EARS-03 appear here as acceptance tests.

How to read it:
- The Glossary (§3) defines every domain noun once. Use those terms exactly.
- Most features realize the user journey in §2.3. Requirements are nested under features and numbered globally (FR-n).
- Inferred items are tagged `[ASSUMPTION]` and indexed in §10. Open questions that block a later phase are marked **Blocks** in §9.

## 1. Vision

Find is a browser-based file explorer for people who need files throughout the workday but rarely remember a file's exact name or where they saved it. It runs in Chrome or Edge on a folder the user chooses. It pairs a clear folder tree with forgiving search: partial names, English typos, and words *inside* text files and text-layer PDFs all count. Results are ranked best first and can be narrowed by type and by date. The date filter understands dates written in file names as well as the file's modified date. A built-in preview lets the user confirm they have the right file.

Everything happens on the user's machine. Find only reads: it never changes, moves, or deletes a file. It uploads nothing, and it forgets everything when the session ends. That makes it safe to use on any work computer, with nothing to install.

The MVP has one job: to show that forgiving, content-aware search gets users to the right file faster than their operating system's search. Version grouping, duplicate detection, and conversational search wait until that is proven.

### 1.1 Product Principles

These principles settle trade-offs that the requirements don't settle.

1. **Never hide the right file.** Ranking, filters, and labels may reorder or narrow the result list, but Find never silently removes a file entry. When filters hide matches, Find says so (FR-12). When a file can't be searched, Find says why (FR-16). A smart feature that hides the right file is worse than no feature.
2. **Search first.** Every MVP feature must help the user find, narrow, or confirm a file. Anything else waits.
3. **Speed and responsiveness.** Name search never waits for indexing, and heavy work never freezes the interface (FR-13, NFR-4).
4. **Transparent results.** Find always shows why a file entry matched (its match kind, FR-11), which date source matched (FR-18), and whether results are still updating (FR-15).
5. **Privacy and integrity by design.** All processing is local, nothing leaves the machine, nothing is written to the user's files, and nothing is kept after the session (NFR-1 to NFR-3).
6. **User control.** Find suggests and the user decides. Find never acts on files.
7. **Simplicity.** A first-time user can choose a folder and find a file without any onboarding or instructions.
8. **Built to extend.** MVP choices must not block the planned next steps: version clusters, more formats, and conversational search.

## 2. Target User

### 2.1 Jobs To Be Done

- **Core job.** "When I need a file I worked with before but only half remember, I want to describe what I *do* remember and see the right file at the top. Then I can get back to work without hunting through folders."
- **Confirm the file.** "Before I open it in another app, I want to confirm it's the right file."
- **Trust.** "I want to be sure the tool can't damage, move, or leak my files, so I can point it at work folders without worrying."

### 2.2 Non-Users (MVP)

- **People who want to organize files** by renaming, moving, or deleting them. Find is read-only.
- **Firefox, Safari, and mobile users.** The folder access Find needs exists only in desktop Chrome and Edge.
- **People whose important content is in scanned documents, Word files, or Excel files.** Name search and the date filter still work for them, but content search does not.
- **People who want always-on, system-wide search across the whole computer.** Find works on one selected folder per session.

### 2.3 Key User Journeys

- **UJ-1. Daniel finds a report he only half remembers.** [ASSUMPTION: UJ-1 was structured from the user's narrative and accepted without explicit line-by-line confirmation]
  - **Persona + context:** Daniel works on several projects on his own computer, and documents pile up across many folders. Midway through the workday he needs a report he worked on earlier. He remembers part of its name, or a phrase from inside it, and roughly when he worked on it. He does not remember the exact name or where he saved it.
  - **Entry state:** Find is open in Chrome or Edge, in the start state. Find remembers nothing from earlier sessions.
  - **Path:**
    1. Daniel clicks **Choose folder**, picks his projects folder, and grants read-only access. The folder tree appears progressively. Name search works immediately, and indexing runs in the background with visible progress.
    2. He types `project reprot`, with a typo.
    3. The result list appears, best match first. Each file entry shows its name, type, location, modified date, and match kind. Content matches are added as indexing continues, and the result list is marked *still updating* until indexing completes.
    4. There are too many results, so he sets a date filter of Sep 1–30, 2026. The result list narrows to file entries whose modified date *or* name date falls in September, and each one shows which date matched.
    5. He selects the most likely file entry, and its preview opens inside Find.
  - **Climax:** The preview shows the phrase he remembered, so he knows this is the right file.
  - **Resolution:** He notes the file's location (shown from the selected folder's name down) and opens it in his usual app.
  - **Edge cases:**
    - **No match:** Find shows "No matching files found" (EARS-03).
    - **PDF without a text layer:** it is labelled **content not searchable**.
    - **Folder not opened:** if the picker closes without granting access, possibly because the browser blocks that folder (OQ-1), Find returns to the start state with a hint to try a subfolder or his MyFiles folder.

## 3. Glossary

*Use these terms exactly, everywhere. Do not introduce synonyms.*

**Session and folder**
- **Start state:** What Find shows when no folder is selected: the **Choose folder** action and a short explanation, with no folder tree, result list, or preview.
- **Selected folder:** The one folder the user chooses in a session. Find reads only this folder and its subfolders.
- **Session:** The period from choosing a selected folder until the tab is closed, the page is reloaded, or a different folder is chosen. All data Find holds lives only for the session.
- **Read-only access:** The browser permission Find requests for the selected folder. It allows reading only.
- **Blocked folder:** A folder the browser refuses to grant read-only access to, such as a system folder or the home folder. Whether Downloads and Desktop are blocked is unverified (OQ-1).
- **MyFiles folder:** The recommended selected folder. It is a folder named `MyFiles` that the user creates and sets as the browser's download location, so every file they download lands in one place Find can open. Find never creates it, because Find is read-only (NFR-2). Find guides the user through the setup (FR-1).
- **Folder tree:** The hierarchical view of the selected folder's subfolders and files. It loads progressively.

**Files**
- **File entry:** A file inside the selected folder as Find knows it: name, type, size, location, modified date, and name date if one is found.
- **Location:** A file entry's path, shown from the selected folder's name downward. Browsers do not reveal the selected folder's full path on disk.
- **Type:** A file entry's category, derived from its extension:
  - **text file:** TXT, MD, CSV, JSON, LOG, and plain-text code files (JS, TS, PY, JAVA, C, CPP, CS, HTML, CSS, XML, YML, YAML, SQL, SH). [ASSUMPTION: extension list]
  - **PDF**
  - **image:** PNG, JPG, JPEG, GIF, WebP.
  - **other:** everything else, including SVG, DOCX, and XLSX in the MVP.
- **Text-layer PDF:** A PDF that contains extractable text. A PDF without a text layer, such as a scanned document, has no searchable content.
- **Modified date:** The file's last-modified timestamp as the browser reports it. It is *not* the date the file was last opened, which browsers cannot see.
- **Name date:** A date found in a file entry's name, such as `invoice_2025-12.pdf` → December 2025. It covers a day, a month, or a whole year, depending on what the name contains (FR-18).
- **Date source:** Where a date comes from: the **modified date** or the **name date**.

**Search**
- **Query:** The text the user types into search. It is split into **query words**.
- **Name token:** One piece of a file entry's name. Names are split at spaces, underscores, hyphens, and dots; at lower-to-upper case changes (`ProjectReport` → `Project`, `Report`); and between letters and digits (`Q3` → `Q`, `3`). Matching ignores case.
- **Name match:** A file entry where *every* query word matches some name token (FR-9, FR-10).
- **Searchable content:** Text extracted from a text file or a text-layer PDF.
- **Content match:** A file entry whose searchable content contains *every* query word (FR-14).
- **Content not searchable:** The label on a file entry whose content was not indexed. It always comes with a reason: no text layer, over the size limit, memory budget reached, failed to read, or format not supported yet (FR-16).
- **Content-only match:** A content match that is not also a name match.
- **Match kind:** The label explaining why a file entry is in the result list. It is one of:
  - **exact:** the whole query appears in the name, ignoring case.
  - **partial:** every query word matches a name token exactly or as its beginning, but the whole query does not appear as typed.
  - **typo:** at least one query word matches a name token only within typo tolerance.
  - **content:** a content-only match.

  A file entry that is both a name match and a content match is labelled with its name match kind.
- **Content index:** The in-memory store of searchable content for the session. It is built during indexing and discarded when the session ends.
- **Indexing:** Building the content index in the background. While it runs, the result list is *still updating*.
- **Filter:** A condition that narrows the result list:
  - **type filter:** one or more types.
  - **date filter:** a date range, applied to one or both date sources.
- **Result list:** The file entries that match the query and every active filter.
- **Rank:** The order of the result list.
  - With a query: exact, then partial, then typo, then content. Ties go to the most recent modified date. [ASSUMPTION: name matches outrank content-only matches]
  - With no query: most recent modified date first.
- **Selected file entry:** The file entry the user has chosen in the result list or the folder tree. There is at most one.
- **Preview:** The read-only view, inside Find, of the selected file entry.

## 4. Features

### 4.1 Folder Access and Session

**Description:** The user starts every session by choosing a selected folder. Find requests read-only access and nothing more. Because Find keeps nothing, ending the session discards everything Find knew. Realizes UJ-1 (entry, and the "folder not opened" edge case).

#### FR-1: Choose a selected folder

The user can choose one folder on their computer as the selected folder. Realizes UJ-1.

**Consequences (testable):**
- Choosing a folder requires an explicit user action: a click on **Choose folder**. Find never opens the folder picker on its own.
- Find requests read-only access. It never requests write access.
- After access is granted, the folder tree begins loading, and name search becomes available.
- The start state offers a short, skippable **Set up MyFiles** guide in three steps:
  1. Create a folder named `MyFiles` inside your Documents folder.
  2. In Chrome or Edge settings, set it as the download location.
  3. Choose it with **Choose folder**.

  [ASSUMPTION: the guide recommends Documents as the location]
- Each session, the user still chooses the folder: Find cannot open MyFiles automatically, because it keeps no folder permission (NFR-3).

#### FR-2: Folder not opened

When the folder picker closes without granting access, Find returns to the start state and helps the user try again. The browser reports a cancelled picker and a blocked folder the same way, so Find treats them the same.

**Consequences (testable):**
- If the picker closes without granting access, Find shows the start state with a hint that is not styled as an error: "Couldn't open that folder. Some system and personal folders can't be opened in the browser. Try a subfolder, or your MyFiles folder." The hint links to the **Set up MyFiles** guide. **Choose folder** is offered again.
- The hint stays until the user chooses a folder successfully.

**Notes:** The MyFiles folder is the chosen answer to blocked folders: browser downloads land in one folder that is never blocked. Files saved by desktop email and chat apps may still land elsewhere, unless the user saves them to MyFiles. See OQ-1.

#### FR-3: Session lifetime

All session data exists only for the current session.

**Consequences (testable):**
- After a page reload, Find shows the start state.
- Choosing a different folder discards the previous folder tree, content index, query, filters, and selected file entry.

#### FR-4: Unsupported browser

When Find runs where folder access is not available, it says so instead of failing silently.

**Consequences (testable):**
- In Firefox, in Safari, and in any mobile browser, Find shows a message that desktop Chrome or Edge is required, and it does not offer **Choose folder**.

### 4.2 Folder Tree and File Details

**Description:** The folder tree shows the selected folder's structure, and the user can browse it at any time, with or without a query. Expanding a folder shows its file entries with their details. Large folders load progressively, so the folder tree is usable before the whole folder has been read.

#### FR-5: Browse the folder tree

The user can expand and collapse folders in the folder tree.

**Consequences (testable):**
- On the reference machine (NFR-4), the first level of the folder tree appears within 1 second of access being granted, for a selected folder of up to 10,000 file entries. [ASSUMPTION]
- Expanding a folder shows its subfolders and file entries. Subfolders are listed before file entries.

#### FR-6: View file details

The user can see each file entry's name, type, size, location, and modified date, plus its name date when one is found.

**Consequences (testable):**
- Size is shown in bytes below 1 KB, then in KB, MB, or GB with one decimal place.
- The modified date is shown with both date and time. The name date is shown at the precision found in the name (day, month, or year).
- The two dates are labelled **Modified** and **Date in name**, so they can never be confused.

#### FR-7: Sort file entries

The user can sort the file entries in an expanded folder of the folder tree, and in the result list, by name, type, size, or modified date, ascending or descending.

**Consequences (testable):**
- In the folder tree, subfolders stay above file entries under every sort.
- In the result list, choosing a sort replaces rank order. The user can return to rank order at any time.

### 4.3 Name Search

**Description:** The user types a query, and Find matches it against the names of every file entry in the selected folder and its subfolders. Search forgives the ways people misremember names: missing words, different separators, wrong case, word order, and small English spelling errors. Results appear in rank order, and each file entry shows its match kind. Name search never waits for indexing. Realizes UJ-1 (steps 2–3).

#### FR-8: Search file names

The user can enter a query and get a result list of file entries whose names match it, from the selected folder and all its subfolders. Realizes UJ-1.

**Consequences (testable):**
- **EARS-01 acceptance:** in a folder containing `report.pdf`, `photo.jpg`, and `report-final.docx`, the query `report` returns `report.pdf` and `report-final.docx`, and not `photo.jpg`.
- Matching ignores case: `REPORT` matches `report.pdf`.
- The result list updates as the user types, without pressing Enter.

#### FR-9: Word-by-word matching

A file entry is a name match when every query word matches a name token, in any order. A query word matches a token exactly, as the token's beginning, or within typo tolerance (FR-10).

**Consequences (testable):**
- `project report` matches `Project_Report_Q3.docx`, `project-report.pdf`, and `ProjectReport.txt`.
- `q3 report` matches `Project_Report_Q3.docx`, whatever the word order.
- `proj rep` matches `Project_Report_Q3.docx`, because both words match token beginnings.
- `project budget` does **not** match `Project_Report_Q3.docx`, because `budget` matches no token.

#### FR-10: Typo tolerance (English)

A query word can match a name token despite small English spelling errors.

**Consequences (testable):**
- `reprot` matches `report.pdf`, and `recipt` matches `receipt.pdf`.
- Query words of 4–7 letters tolerate 1 edit. Words of 8 or more letters tolerate 2 edits. Words of 3 letters or fewer must match exactly or as a token's beginning. [ASSUMPTION]
- Names in non-Latin scripts get exact and token-beginning matching only, with no typo tolerance.

#### FR-11: Ranked results that explain themselves

The result list appears in rank order (§3 Rank), and every file entry in it shows its match kind.

**Consequences (testable):**
- Each file entry in the result list is labelled **exact**, **partial**, **typo**, or **content**.
- `project reprot` returns `Project_Report.pdf` labelled **typo**, because one query word matched only within typo tolerance.
- For the query `report`, a file named `report.pdf` (exact) ranks above `Q3_Reports.pdf` (partial), whatever their modified dates.

#### FR-12: Empty, no-match, and filtered-out states

Find always tells the user why the result list is empty or short, so the right file is never hidden without explanation.

**Consequences (testable):**
- **EARS-03 acceptance:** in a folder containing `report.pdf` and `photo.jpg`, after indexing completes, the query `xyz123` shows no file entries and the message "No matching files found".
- While indexing is running and there are no matches yet, Find shows "No matches yet. Still searching file contents…" instead of "No matching files found".
- When active filters hide matches, Find shows how many, for example "Filters hide 12 matches", with a **Clear filters** action. This applies whether the visible result list is empty or not.
- With an empty query and no active filter, Find shows the folder tree, not a result list.
- With an empty query and any active filter, the result list contains every file entry that passes the filters, in rank order (most recent modified date first).

### 4.4 Content Search

**Description:** Once a folder is selected, Find reads the searchable content of text files and text-layer PDFs in the background and builds the content index. Content matches join the result list as they are found. The user always knows whether results are complete, and which files were not searched. Realizes UJ-1 (steps 1 and 3, and the text-layer edge case).

#### FR-13: Background indexing

Find builds the content index in the background after a folder is selected. The user can keep browsing and searching while it runs.

**Consequences (testable):**
- Indexing starts automatically after access is granted.
- Find shows progress as the number of files indexed out of the total number of text files and PDFs, for example "Indexing 340 / 1,200 files".
- The user can cancel indexing. Name search and the folder tree keep working after a cancel, and the result list states that content search is incomplete.
- No interaction with the folder tree, search box, or result list freezes for more than 100 ms during indexing. [ASSUMPTION]

#### FR-14: Search inside files

The query also matches the searchable content of indexed file entries.

**Consequences (testable):**
- A file entry is a content match when every query word appears in its searchable content as a whole word or as a word's beginning, ignoring case. Content matching has no typo tolerance in the MVP. [ASSUMPTION: architecture should treat this rule as fixed for the MVP]
- Each content match shows a snippet of up to 160 characters around the first match, with the matched words highlighted. [ASSUMPTION]

#### FR-15: Results that are still updating

While indexing runs, the result list says it is incomplete.

**Consequences (testable):**
- While indexing runs, the result list shows a *still updating* indicator. New content matches are inserted in rank order without changing the user's scroll position or the selected file entry.
- When indexing completes, the indicator disappears.

#### FR-16: Files whose content is not searched

Find tells the user which file entries were not content-searched, and why.

**Consequences (testable):**
- Each of these is labelled **content not searchable**, with its reason:
  - a PDF without a text layer;
  - a file over the 10 MB per-file limit [ASSUMPTION];
  - a file skipped because the content index reached its memory budget (NFR-4);
  - a file that failed to read.
- DOCX and XLSX files are labelled **content not searchable**, with the reason "format not supported yet", so users never assume Word or Excel content was searched.
- A file that fails to read never stops indexing of the other files.
- Find shows which formats content search covers: text files and text-layer PDFs.

### 4.5 Filters

**Description:** When a query returns too many results, or when the user remembers *what kind* of file it was or *roughly when* they worked on it, filters narrow the result list. Filters work with or without a query (FR-12). Realizes UJ-1 (step 4).

#### FR-17: Type filter

The user can limit the result list to one or more types: text file, PDF, image, other.

**Consequences (testable):**
- Selecting PDF shows only PDF file entries. Selecting PDF and image shows both.
- Clearing the type filter restores the unfiltered result list.

#### FR-18: Smart date filter

The user can limit the result list to file entries whose date falls in a date range, using the modified date, the name date, or both. Realizes UJ-1 (step 4).

**Consequences (testable):**
- The user sets a range (from and to, inclusive), or picks a whole year. Either end of a range may be left empty.
- The user chooses the date source: **Modified or date in name** (default), **Modified only**, or **Date in name only**.
- With the default, a file entry passes when its modified date *or* its name date falls in the range. Example: `invoice_2025-12.pdf`, modified 10 Jan 2026, passes the filter "2025".
- A name date passes when any part of it falls in the range. A year-only name date such as `report_2024.pdf` passes any range that includes part of 2024.
- Each file entry that passes shows which date source matched: **Modified**, **Date in name**, or both, with the matching date.
- Recognized name-date formats are year first: `YYYY-MM-DD`, `YYYY_MM_DD`, `YYYY.MM.DD`, `YYYYMMDD`, `YYYY-MM`, `YYYY_MM`, and a standalone year from 1990 to 2099. Formats with the day or month first, such as `03-04-2025`, are not recognized, because they are ambiguous. [ASSUMPTION]
- The date filter combines with the query and the type filter, so all of them must match.

### 4.6 Locate and Preview

**Description:** The user confirms they have the right file without leaving Find. They can see where it lives in the folder tree and preview its contents, read-only. Realizes UJ-1 (step 5, climax, and resolution).

#### FR-19: Show a file entry in the folder tree

The user can jump from a file entry in the result list to its place in the folder tree. Confirmed by the user.

**Consequences (testable):**
- Choosing **Show in folder** expands the folder tree to the file entry's folder and highlights the file entry.
- Each file entry in the result list shows its location.

#### FR-20: Preview a file

The user can preview the selected file entry inside Find, read-only.

**Consequences (testable):**
- **Text files** display as plain text. Previews of files larger than 1 MB are cut off, with a notice. [ASSUMPTION]
- **PDFs** display page by page.
- **Images** display at fit-to-panel size.
- For the type **other**, Find shows the file details and "Preview not available for this file type".
- When the selected file entry is a content match, the preview scrolls to the first matched words. For a PDF, it opens on that page.
- **EARS-02 acceptance:** with a folder selected and no selected file entry, the preview shows "Select a file to preview".

## 5. Cross-Cutting Non-Functional Requirements and Guardrails

- **NFR-1 Privacy: no network after load.** After the initial page load, Find makes zero network requests for the rest of the session. Everything it needs, including the PDF library, is loaded up front. Find includes no analytics or telemetry. *Verify:* browser developer tools show no requests after load during a full session.
- **NFR-2 Read-only.** Find requests read-only access only, and never creates, modifies, renames, moves, or deletes a file. *Verify:* the permission prompt asks for read access only, and every file's modified date and size are unchanged after a full session.
- **NFR-3 No persistence.** Find writes no file content, file metadata, folder permission, query, or filter to any browser storage (local storage, IndexedDB, Cache Storage, or cookies). *Verify:* storage for Find's site is empty after a session.
- **NFR-4 Performance and capacity.** The reference machine is a laptop with 4 cores, 8 GB RAM, and an SSD, running current Chrome. [ASSUMPTION] These are design targets for architecture:
  - For a selected folder of up to 10,000 file entries, name results appear within 200 ms of the last keystroke. [ASSUMPTION]
  - For up to 50,000 file entries, name results appear within 1 second, and Find shows a notice that the folder is large. Folders larger than that are not supported in the MVP, and Find says so. [ASSUMPTION]
  - Indexing of 1,000 mixed text files and text-layer PDFs completes within 60 seconds. [ASSUMPTION]
  - The content index uses at most 500 MB of memory. Files beyond that budget are labelled **content not searchable** (FR-16). [ASSUMPTION]
- **NFR-5 Safe previews.** Previewed content is shown as inert content: it can never run scripts or reach the network.
- **NFR-6 Accessibility.** Search, the result list, filters, the folder tree, and the preview are fully usable by keyboard, and meet WCAG 2.1 AA. [ASSUMPTION]
- **NFR-7 Platform.** Find supports the current and previous major versions of desktop Chrome and Edge on Windows, macOS, and Linux. The interface is in English.
- **NFR-8 Testable rules.** Matching, ranking, filtering, name-date detection, and indexing rules can be unit-tested without rendering the UI, with at least 90% coverage. This is a stated project goal (portfolio).

## 6. Non-Goals (Explicit)

- Find is **not a file manager**. It never changes files, and it won't without a deliberate future decision and explicit user permission.
- Find is **not system-wide search**. It has no background service and no always-on index, and it covers one selected folder per session.
- Find is **not a document editor** or full document viewer. The preview exists to confirm the file, not to work in it.
- Find is **not a cloud product**. It has no accounts, sync, sharing, or server.
- Find **does not remember**. It keeps no history, saved searches, or recent files across sessions.

## 7. MVP Scope

### 7.1 In Scope

- Choosing a folder, recovering when a folder is not opened, session lifetime, and unsupported-browser handling (FR-1 to FR-4).
- Folder tree, file details, and sorting (FR-5 to FR-7).
- Name search with word-by-word, token-beginning, and English typo matching, plus ranked and labelled results with clear empty and filtered-out states (FR-8 to FR-12).
- Content search over text files and text-layer PDFs, with background indexing (FR-13 to FR-16).
- Type filter, and a smart date filter over modified dates and name dates (FR-17, FR-18).
- Showing a file entry in the folder tree, and previews of text files, PDFs, and images (FR-19, FR-20).

### 7.2 Out of Scope for MVP

- **Version clusters and version comparison.** Deferred: the MVP proves search first. [NOTE FOR PM] This was the brief's strongest differentiator. Revisit it first after the MVP.
- **Duplicate detection and cleanup recommendations.** Future.
- **OCR, and content search in DOCX and XLSX.** Future, in this order: DOCX, then XLSX, then OCR.
- **A list/thumbnail view switch.** [ASSUMPTION: cut, per the brief]
- **Ranking by "recent usage".** Not possible: browsers cannot see when a file was opened, and Find stores nothing.
- **Typo tolerance in content search, and typo tolerance for non-English text.**
- **Name dates written day-first or month-first**, such as `03-04-2025`, because they are ambiguous.
- **Full on-disk paths.** Browsers don't reveal them. Locations start at the selected folder's name.
- **Firefox, Safari, and mobile.**
- **More than one selected folder per session.**
- **Conversational or AI search.** Vision.

## 8. Success Metrics

**Primary**
- **SM-1: Faster to the right file.** In a moderated test, 5 users each do 5 tasks with half-remembered cues, on a seeded folder. At least 2 of the 5 tasks are solvable only through content or a name date. Each user does the same tasks in Find and in their own operating system's search, in counterbalanced order. Target: median time to the correct file of 30 seconds or less in Find, and at least 3 of 5 users faster in Find. [ASSUMPTION] Validates FR-8 to FR-20.
- **SM-2: Forgiving name search works.** On a fixture set of queries with one typo or a partial name, the target file appears in the top 5 at least 90% of the time. Validates FR-9 to FR-11.
- **SM-3: Content search works.** On a fixture set of queries whose words appear only inside the target file, the target appears in the top 5 at least 90% of the time once indexing completes. Validates FR-14.

**Secondary**
- **SM-4: Name search latency.** 200 ms or less at 10,000 file entries on the reference machine. Validates FR-8, NFR-4.
- **SM-5: Indexing throughput.** 1,000 mixed files indexed within 60 seconds, with no freeze over 100 ms. Validates FR-13, NFR-4.
- **SM-6: Privacy and integrity hold.** An automated check of a full session, measured after the initial load, counts zero network requests, zero storage writes, and zero changed files. Validates NFR-1 to NFR-3.
- **SM-7: Name dates are detected correctly.** On a fixture set of file names, every recognized format yields the correct name date, and day-first or month-first names yield none. Validates FR-18.
- **SM-8: Portfolio quality.** Rule coverage of at least 90% (NFR-8), and a complete BMAD trail: brief, PRD, architecture, stories.

**Counter-metrics (do not optimize)**
- **SM-C1: Wrong files in the top 5.** Loosening typo tolerance raises SM-2 but floods the result list with wrong files. Keep the false-match rate in the top 5 on the SM-2 fixture set at 20% or less. [ASSUMPTION] Counterbalances SM-2.
- **SM-C2: Silently skipped files.** Never speed up indexing by quietly skipping files. Every file entry left unindexed must be labelled (FR-16). Counterbalances SM-5.
- **SM-C3: Over-eager name dates.** Detecting more number patterns as dates would make more files pass date filters, but wrongly. A project code like `ticket_20251` must not become a date. Counterbalances SM-7.

## 9. Open Questions

- **OQ-1. Blocked folders.** *Informs FR-1 and FR-2.* The decision is the MyFiles folder. Still to verify in the architecture spike:
  - Can Chrome and Edge open a `MyFiles` folder inside Documents on Windows, macOS, and Linux?
  - Are Downloads and Desktop blocked, for users who skip the setup?
  - Does drag-and-drop get around the block?

  This no longer blocks UX.
- **OQ-2. Real folder sizes.** *Informs NFR-4.* How many files does a typical user's work folder hold? Until this is answered, the NFR-4 numbers are fixed design targets.
- **OQ-3. Name rank vs content rank.** Should a content-only match ever beat an old name match? For example, the query `budget` against `notes.txt` (with "budget" inside) and `budget_2024.xlsx`. Validate in the SM-1 test.
- **OQ-4. Typo limits.** Are the FR-10 edit limits right? Tune them against SM-2 and SM-C1.
- **OQ-5. Daily folder re-selection.** Users must choose the folder again every session. If the SM-1 test shows this is a deal-breaker, reconsider remembering *only the folder permission*, not file data. That would be an explicit exception to NFR-3.
- **OQ-6. Show in folder.** *Resolved:* FR-19 confirmed in the MVP.
- **OQ-7. Name.** Check the availability of "Find" before any public release.
- **OQ-8. Brief out of sync.** *Resolved:* the brief was updated to match this PRD.

## 10. Assumptions Index

Defaults that architecture should treat as fixed design targets: FR-14, NFR-4. All others can be tuned later.

- §2.3 UJ-1 was structured from the user's narrative and accepted without line-by-line confirmation.
- §3 Type: the extension lists for each type.
- §3 Rank, FR-11: name matches outrank content-only matches.
- FR-5: the first level of the folder tree within 1 second.
- FR-10: typo edit limits.
- FR-13: no freeze over 100 ms.
- FR-14: content matching rules and the 160-character snippet.
- FR-16: the 10 MB per-file limit.
- FR-18: recognized name-date formats.
- FR-1: the MyFiles guide recommends Documents as the location.
- FR-20: the 1 MB text preview cut-off.
- NFR-4: the reference machine, scale limits, timings, and the 500 MB memory budget.
- NFR-6: WCAG 2.1 AA.
- §7.2: the list/thumbnail switch is cut.
- SM-1 and SM-C1 targets.
