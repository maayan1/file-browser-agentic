---
title: "PRD: Find"
status: final
created: 2026-10-25
updated: 2026-10-25
---

# PRD: Find

> **Core principle:** Users should not need to remember a file's exact name or location to find it.
>
> **Scope reduced (2026-10-25):** the MVP is a focused, **filename-only** file finder. Requirements outside it keep their IDs, and §11 quotes their last approved text verbatim. Nothing was renumbered.

## 0. Document Purpose

This PRD is for whoever designs, architects, and builds Find, including the downstream BMAD workflows: UX, architecture, and epics and stories.

It builds on these inputs and does not repeat them:
- The product brief: `_bmad-output/planning-artifacts/briefs/brief-file-brwsr-2026-10-25/`.
- The EARS requirements: `docs/product/earsRequirments`. Their acceptance tests appear in the FRs below.

How to read it:
- The Glossary (§3) defines every domain noun once. Use those terms exactly.
- MVP requirements are in §4 and §5. Deferred ones are in §11, with their original IDs and text.
- Inferred items are tagged `[ASSUMPTION]` and indexed in §10.
- An open question in §9 that blocks a later phase is marked **Blocks**. None currently does.

**Requirement status**

| Status | IDs |
|---|---|
| MVP, unchanged | FR-4, FR-17; NFR-2, NFR-3, NFR-7 |
| MVP, changed (prior text in §11.2) | FR-1, FR-2, FR-3, FR-6, FR-8, FR-9, FR-12, FR-15, FR-21; NFR-1, NFR-4, NFR-5, NFR-6, NFR-8 |
| Deferred (§11) | FR-5, FR-7, FR-10, FR-11, FR-13, FR-14, FR-16, FR-18, FR-19, FR-20, FR-22 |

**EARS traceability**

| EARS | Status | Carried by | Acceptance test |
|---|---|---|---|
| EARS-01 (name contains search term) | Unchanged. Satisfied by FR-9, with two noted edge cases: a query of only separators counts as empty, and multi-word queries match words in any order (each word must be contained) | FR-8, FR-9 | FR-8 |
| EARS-02 ("Select a file to preview") | **Deferred** with preview (FR-20, §11). Text unchanged | FR-20 (deferred) | §11 FR-20 |
| EARS-03 ("No matching files found") | Revised (rev. 3, 2026-10-25): listing only. Earlier versions are kept in the EARS file's revision history | FR-12 | FR-12 |
| EARS-04 ("No matches yet. Still searching…") | Revised (rev. 1, 2026-10-25): listing only | FR-12 | FR-12 |

## 1. Vision

Find is a browser-based file finder for people who need files throughout the workday but rarely remember a file's exact name or where they saved it. It runs in Chrome or Edge on a folder the user chooses. The user types any part of a file name, in any order, and Find lists every matching file with its type, location, and modified date. A type filter narrows the list, and **Copy location** hands the file off to File Explorer or Finder.

Everything happens on the user's machine. Find only reads: it never changes, moves, or deletes a file. It uploads nothing, and it forgets everything when the session ends. That makes it safe to use on any work computer, with nothing to install.

The MVP has one job: to prove a clean, private, filename-only finder that users can trust and that is easy to extend. Typo tolerance, ranking, content search, smart dates, previews, and version grouping are deferred (§11) and build on this foundation.

### 1.1 Product Principles

These principles settle trade-offs that the requirements don't settle.

1. **Never hide the right file.** Filters may narrow the result list, but Find never silently removes a file entry. When filters hide matches, Find says so. When listing hasn't finished, Find never claims "no match" (FR-12).
2. **Search first.** Every MVP feature must help the user find, narrow, or hand off a file. Anything else waits.
3. **Speed and responsiveness.** Listing a large folder never freezes search or the interface (NFR-4).
4. **Transparent results.** Find always shows whether results are still updating (FR-15), and why the result list is empty (FR-12).
5. **Privacy and integrity by design.** All processing is local, nothing leaves the machine, nothing is written to the user's files, and nothing is kept after the session (NFR-1 to NFR-3).
6. **User control.** Find suggests and the user decides. Find never acts on files.
7. **Simplicity.** A first-time user can choose a folder and find a file without any onboarding.
8. **Built to extend.** MVP choices must not block the deferred features in §11.

## 2. Target User

### 2.1 Jobs To Be Done

- **Core job.** "When I need a file I worked with before but only half remember its name, I want to type the part I *do* remember and see the file in the list. Then I can get back to work without hunting through folders."
- **Hand off.** "Once I've found it, I want to find it in my usual tools and open it there."
- **Trust.** "I want to be sure the tool can't damage, move, or leak my files, so I can point it at work folders without worrying."

### 2.2 Non-Users (MVP)

- **People who want to organize files** by renaming, moving, or deleting them. Find is read-only.
- **Firefox, Safari, and mobile users.** The folder access Find needs exists only in desktop Chrome and Edge.
- **People who remember what is inside a file, not its name.** Content search is deferred (§11).
- **People who want always-on, system-wide search across the whole computer.** Find works on one selected folder per session.

### 2.3 Key User Journeys

- **UJ-1. Daniel finds a PDF report by part of its name.** [ASSUMPTION: UJ-1 was structured from the user's narrative and accepted without explicit line-by-line confirmation]
  - **Persona + context:** Daniel works on several projects, and the documents he receives pile up across many folders. Midway through the workday he needs a PDF report he worked with earlier. He remembers that its name has "project" and "report" in it, and that it's a PDF. He does not remember the exact name or where he saved it.
  - **Entry state:** Find is open in Chrome or Edge, in the start state. Find remembers nothing from earlier sessions.
  - **Path:**
    1. Daniel clicks **Choose folder**, picks his MyFiles folder, and grants read-only access. Listing starts, and name search covers file entries as listing reaches them.
    2. He types `report project`.
    3. The result list shows every file whose name contains both words, in any order, sorted alphabetically. Each file entry shows its name, type, location, and modified date. The list is marked *still updating* until listing completes.
    4. There are too many results, so he selects the type filter **PDF**. The list narrows, and a notice says how many matches the filter hides.
  - **Climax:** He recognizes `Project_Report_Sep.pdf` in `MyFiles/Clients/Acme/` by its name, location, and modified date.
  - **Resolution:** He clicks **Copy location** (`MyFiles/Clients/Acme/Project_Report_Sep.pdf`), opens his MyFiles folder in File Explorer, follows the path to the file, and opens it in Acrobat.
  - **Edge cases:**
    - **No match:** while listing is still running, Find shows "No matches yet. Still searching…" (EARS-04). Once listing has finished, Find shows "No matching files found" (EARS-03).
    - **The filter hides everything:** Find says how many matches the filter hides, and offers **Clear filters**.
    - **Folder not opened:** if the picker closes without granting access, possibly because the browser blocks that folder (OQ-1), Find returns to the start state with a hint to try a subfolder or his MyFiles folder.

## 3. Glossary

*Use these terms exactly, everywhere. Do not introduce synonyms. Terms used only by deferred requirements are listed in §11.*

**Session and folder**
- **Start state:** What Find shows when no folder is selected: the **Choose folder** action, the MyFiles hint, and a short explanation, with no result list.
- **Selected folder:** The one folder the user chooses in a session. Find reads only this folder and its subfolders.
- **Session:** The period from choosing a selected folder until the tab is closed, the page is reloaded, or a different folder is chosen. All data Find holds lives only for the session.
- **Read-only access:** The browser permission Find requests for the selected folder. It allows reading only.
- **Blocked folder:** A folder the browser refuses to grant read-only access to. In Chromium, this includes the home, Desktop, Documents, and Downloads folders themselves, but not the folders inside them (OQ-1).
- **MyFiles folder:** The recommended selected folder: a folder named `MyFiles`, in a local folder that isn't cloud-synced, which the user creates and sets as the browser's download location. Find never creates it, because Find is read-only (NFR-2). Find mentions it in a short hint (FR-1, FR-2).
- **Listing:** Reading the names and details of every file entry in the selected folder and its subfolders. It runs once per session. Name search covers file entries as listing reaches them.
  - Hidden files and folders, such as names starting with `.`, are included.
  - Only operating-system clutter is skipped: `.DS_Store`, `Thumbs.db`, and `desktop.ini`.
  - Listing ends in one of three ways:
    - **completed:** every folder was attempted; unreadable folders are counted, not retried;
    - **capped:** it stopped at the 50,000-entry limit (NFR-4);
    - **stopped:** read access was lost.
  - A subfolder that can't be read is counted as an **unreadable folder**, and listing continues.

**Files**
- **File entry:** A file inside the selected folder as Find knows it: name, type, location, and modified date.
- **Location:** A file entry's path, shown from the selected folder's name downward, for example `MyFiles/Clients/Acme/report.pdf`. Browsers do not reveal the selected folder's full path on disk.
- **Type:** A file entry's category, derived from its extension:
  - **text file:** TXT, MD, CSV, JSON, LOG, and plain-text code files (JS, TS, PY, JAVA, C, CPP, CS, HTML, CSS, XML, YML, YAML, SQL, SH). [ASSUMPTION: extension list]
  - **PDF**
  - **image:** PNG, JPG, JPEG, GIF, WebP.
  - **other:** everything else, including SVG, DOCX, and XLSX.
- **Modified date:** The file's last-modified timestamp as the browser reports it. It is *not* the date the file was last opened, which browsers cannot see.

**Search**
- **Query:** The text the user types into search. It is split into **query words** at spaces, underscores, hyphens, and dots. Empty pieces are ignored, so a query of only separators is an empty query.
- **Name match:** A file entry whose name contains *every* query word as a substring, in any order, ignoring case, with names and queries in the same Unicode form (NFC). A name also matches when it does so with its separators removed (FR-9).
- **Filter:** A condition that narrows the result list. In the MVP, the only filter is the **type filter**: one or more types.
- **Result list:** The file entries that match the query and every active filter.
- **Result order:** Alphabetical by name, ignoring case, then by location for equal names (FR-8).

## 4. Features

### 4.1 Folder Access and Session

**Description:** The user starts every session by choosing a selected folder. Find requests read-only access and nothing more. Because Find keeps nothing, ending the session discards everything Find knew. Realizes UJ-1 (entry, and the "folder not opened" edge case).

#### FR-1: Choose a selected folder *(MVP, simplified)*

The user can choose one folder on their computer as the selected folder. Realizes UJ-1.

**Consequences (testable):**
- Choosing a folder requires an explicit user action: a click on **Choose folder**. Find never opens the folder picker on its own.
- Find requests read-only access. It never requests write access.
- After access is granted, listing starts, and name search covers file entries as listing reaches them.
- The start state shows a short MyFiles hint: "Tip: browsers can't open some system folders, such as Downloads itself. Create a `MyFiles` folder (for example `C:\MyFiles` or `~/MyFiles`), save your downloads there, and choose it here." [ASSUMPTION: wording and suggested locations; locations to be verified by OQ-1]
- Each session, the user still chooses the folder. Find cannot open it automatically, because it keeps no folder permission (NFR-3).

*Deferred from the original FR-1 (§11):* the three-step **Set up MyFiles** guide.

#### FR-2: Folder not opened

When the folder picker closes without granting access, Find helps the user try again. If no folder is open, Find returns to the start state. The browser reports a cancelled picker and a blocked folder the same way, so Find treats them the same.

**Consequences (testable):**
- If no folder is open and the picker closes without granting access, Find shows the start state with a hint that is not styled as an error: "Couldn't open that folder. Some system and personal folders can't be opened in the browser. Try a subfolder, or your MyFiles folder." **Choose folder** is offered again.
- The hint stays until the user chooses a folder successfully, which clears it.
- If a folder is already open when the picker closes without granting access, Find keeps the current session and its results, and shows the same hint above the results. [CF-10, decided 2026-10-26]

**Notes:** The MyFiles folder is the chosen answer to blocked folders. Files saved by desktop email and chat apps may still land elsewhere, unless the user saves them to MyFiles. See OQ-1.

#### FR-3: Session lifetime

All session data exists only for the current session.

**Consequences (testable):**
- After a page reload, Find shows the start state.
- Choosing a different folder discards the previous file entries, query, and filters.

#### FR-4: Unsupported browser

When Find runs where folder access is not available, it says so instead of failing silently.

**Consequences (testable):**
- In Firefox, in Safari, and in any mobile browser, Find shows a message that desktop Chrome or Edge is required, and it does not offer **Choose folder**.

### 4.2 File Details

#### FR-6: View file details *(MVP, simplified)*

The user can see each file entry's name, type, location, and modified date in the result list.

**Consequences (testable):**
- The modified date is shown with both date and time.
- If a file entry's details can't be read, it is still listed, with the missing details shown as "unavailable".

*Deferred from the original FR-6 (§11):* size, and the name date.

### 4.3 Name Search

**Description:** The user types a query, and Find matches it against the names of every file entry in the selected folder and its subfolders. Search forgives missing words, different separators, wrong case, and word order. Realizes UJ-1 (steps 2–3).

#### FR-8: Search file names

The user can enter a query and get a result list of file entries whose names match it, from the selected folder and all its subfolders. Realizes UJ-1 and EARS-01.

**Consequences (testable):**
- **EARS-01 acceptance:** in a folder containing `report.pdf`, `photo.jpg`, and `report-final.docx`, the query `report` returns `report.pdf` and `report-final.docx`, and not `photo.jpg`.
- Matching ignores case: `REPORT` matches `report.pdf`.
- The result list updates as the user types, without pressing Enter.
- The result list is in result order: alphabetical by name ignoring case, then by location. Example: `a.pdf` in `MyFiles/x/` comes before `B.pdf` in `MyFiles/a/`, and two files named `a.pdf` are ordered by location.

#### FR-9: Partial, word-order-free matching *(MVP, simplified)*

A file entry is a name match when its name contains every query word as a substring, in any order, ignoring case.

**Consequences (testable):**
- `project report` matches `Project_Report_Q3.docx`, `project-report.pdf`, and `ProjectReport.txt`.
- `report q3` matches `Q3_Report_final.pdf`, whatever the word order.
- `report-final` matches `report-final.docx`, because the query splits into `report` and `final`.
- `port` matches `report.pdf`, and `report` matches `annualreport2025.pdf`.
- `project budget` does **not** match `Project_Report_Q3.docx`, because `budget` doesn't appear in the name.
- `projectreport` matches `Project_Report_Q3.docx`, because the name also matches with its separators (spaces, `_`, `-`, `.`) removed.
- Names and queries are compared in the same Unicode normalization form (NFC). So `café` typed on Windows matches `café.txt` stored by macOS in decomposed form.
- The match runs against the full file name, including the extension. So `pdf` matches `report.pdf`.
- **Edge cases (EARS-01):** a query made only of separators is an empty query, and a multi-word query matches its words in any order.

*Deferred from the original FR-9 (§11):* tokens, the match kinds, and the 3-character substring floor.

#### FR-12: Empty, no-match, and filtered-out states *(MVP, simplified)*

Find always tells the user why the result list is empty or short, and never claims "no match" before listing has finished. Realizes EARS-03 and EARS-04.

**Consequences (testable):**
- **EARS-03 acceptance:** in a folder containing `report.pdf` and `photo.jpg`, after listing completes, the query `xyz123` shows no file entries and the message "No matching files found".
- **EARS-04 acceptance:** with the same folder, while listing is in progress, the query `xyz123` shows no file entries and the message "No matches yet. Still searching…". It changes to "No matching files found" once listing completes.
- When the type filter hides matches, Find shows how many, for example "Filters hide 12 matches", with a **Clear filters** action. It applies whether the visible result list is empty or not.
  - When the visible list is empty, this message replaces the EARS-03 and EARS-04 messages. A file hidden by the filter still matches the name, so saying "No matching files found" would be false.
  - When the filter hides nothing, no notice is shown.
- With an empty query and no active filter, Find shows the prompt "Type part of a file name to search", not a result list. [ASSUMPTION: wording]
- With an empty query and an active filter, the result list contains every file entry that passes the filter, in result order. The notice counts the file entries the filter excludes, and says "files" rather than "matches" (for example "Filters hide 40 files"). If no file entry has been listed yet while listing runs, Find shows "No matches yet. Still searching…" (the same message as EARS-04, outside its scope because the query is empty).
- When listing completes with no file entries at all and no unreadable folders, Find shows "This folder has no files" instead of any search message. If some folders couldn't be read, the unreadable-folders notice is shown instead.
- **Capped listing (NFR-4):** while the listing is capped, Find always shows "This folder has more than 50,000 files. Find searched the first 50,000. Choose a smaller folder." With no matches, Find shows "No matching files found in the first 50,000 files". EARS-03's message never appears on its own, because listing did not complete.
- **Unreadable folders:** when any subfolder couldn't be read, Find shows the notice "N folders couldn't be read" alongside the results. With no matches after listing completes, Find shows "No matching files found in the folders Find could read" (the count appears only in the notice). This is the EARS-03 message, qualified because some folders couldn't be read. EARS tests assert the exact text of each variant.
- **Empty query, nothing to show:** when the query is empty and the result list is empty after listing has ended, Find shows the prompt "Type part of a file name to search", never a "No matching files found" message. The cap and unreadable-folder notices stay visible. [NC-C, decided 2026-10-26]
- **Access lost:** if read access is lost during the session, Find returns to the start state with the FR-2 hint (listing *stopped*).

*Deferred from the original FR-12 (§11):* indexing and cancel states.

#### FR-15: Results that are still updating *(MVP, simplified)*

While listing runs, the result list says it is incomplete.

**Consequences (testable):**
- While listing runs, the result list shows a *still updating* indicator. New matches are inserted in result order without moving the rows the user is looking at: the top visible row stays in place.
- When listing completes, the indicator disappears.

*Deferred from the original FR-15 (§11):* the indicator also covering indexing, and keeping the selected file entry.

### 4.4 Filters

#### FR-17: Type filter

The user can limit the result list to one or more types: text file, PDF, image, other.

**Consequences (testable):**
- Selecting PDF shows only PDF file entries. Selecting PDF and image shows both.
- Clearing the type filter restores the unfiltered result list.

### 4.5 Hand Off

**Description:** Browsers cannot launch desktop apps or reveal full disk paths. So once the user finds a file, Find copies its location for them to find and open it in File Explorer or Finder. Realizes UJ-1 (resolution).

#### FR-21: Copy a file's location *(MVP, simplified)*

The user can copy a file entry's location to find it in other tools.

**Consequences (testable):**
- **Copy location** is available on every file entry and copies its location (for example `MyFiles/Clients/Acme/report.pdf`) to the clipboard, with a short confirmation.
- **Copy location** is usable by keyboard (NFR-6).

*Deferred from the original FR-21 (§11):* opening files in a new browser tab.

## 5. Cross-Cutting Non-Functional Requirements and Guardrails

- **NFR-1 Privacy: no network after load.** *(simplified)* After the initial page load, Find itself makes zero network requests for the rest of the session. Everything it needs is loaded up front. Find includes no analytics or telemetry. *Verify:* browser developer tools show no requests after load during a full session.
- **NFR-2 Read-only.** Find requests read-only access only, and never creates, modifies, renames, moves, or deletes a file. *Verify:* the permission prompt asks for read access only, and every file's modified date and size are unchanged after a full session.
- **NFR-3 No persistence.** Find writes no file content, file metadata, folder permission, query, or filter to any browser storage (local storage, session storage, IndexedDB, Cache Storage, the origin private file system, or cookies). *Verify:* storage for Find's site is empty after a session.
- **NFR-4 Performance and capacity.** *(simplified)* The reference machine is a laptop with 4 cores, 8 GB RAM, and an SSD, running current Chrome. [ASSUMPTION] These are design targets for architecture:
  - For a selected folder of up to 10,000 file entries, name results appear within 200 ms of the last keystroke, including while listing runs. [ASSUMPTION]
  - For up to 50,000 file entries, name results appear within 1 second. Listing stops at 50,000 file entries (capped), and Find says so (FR-12). [ASSUMPTION]
  - No interaction freezes for more than 100 ms during listing. [ASSUMPTION]
- **NFR-5 Inert rendering.** *(simplified)* File names, locations, and any other text that comes from the user's files are displayed as inert text: they can never run scripts or reach the network.
- **NFR-6 Accessibility.** *(simplified)* Choosing a folder, search, the result list, the type filter, and **Copy location** are fully usable by keyboard, and meet WCAG 2.1 AA. [ASSUMPTION]
- **NFR-7 Platform.** Find supports the current and previous major versions of desktop Chrome and Edge on Windows, macOS, and Linux. The interface is in English.
- **NFR-8 Testable rules.** *(simplified)* Matching, filtering, result ordering, and message selection can be unit-tested without rendering the UI, with at least 90% coverage. This is a stated project goal (portfolio).

## 6. Non-Goals (Explicit)

- Find is **not a file manager**. It never changes files, and it won't without a deliberate future decision and explicit user permission.
- Find is **not system-wide search**. It has no background service and no always-on index, and it covers one selected folder per session.
- Find is **not a document viewer**. The MVP has no preview (§11, FR-20).
- Find is **not a cloud product**. It has no accounts, sync, sharing, or server.
- Find **does not remember**. It keeps no history, saved searches, or recent files across sessions.

## 7. MVP Scope

### 7.1 In Scope

- Choosing a folder with read-only access, a MyFiles hint, recovering when a folder is not opened, session lifetime, and unsupported-browser handling (FR-1 to FR-4).
- File details: name, type, location, and modified date (FR-6).
- Name search: case-insensitive, partial, and word-order-free, in alphabetical result order, with clear no-match, still-updating, and filtered-out states (FR-8, FR-9, FR-12, FR-15).
- Type filter (FR-17).
- Copy location (FR-21).
- Local-only processing, read-only access, no persistent storage (NFR-1 to NFR-3).

### 7.2 Out of Scope for MVP

- **Everything in §11 Deferred Requirements:**
  - typo tolerance, ranking, and match kinds;
  - content search and indexing;
  - previews and opening files in a tab;
  - smart date filtering and name dates;
  - the folder tree, Show in folder, and sorting;
  - Rescan.
- **Content search in DOCX and XLSX, and OCR.** After content search itself (§11).
- **Version clusters and version comparison.** [NOTE FOR PM] This was the brief's strongest differentiator. Revisit it after the deferred search features.
- **Duplicate detection and cleanup recommendations.** Future.
- **Launching desktop apps, and full on-disk paths.** Browsers can't do either. Find copies the location (FR-21).
- **Detecting cloud-only files.** Find reads only names and dates in the MVP, so this matters less (OQ-9).
- **Automatic change detection.** The user chooses the folder again to refresh.
- **A list/thumbnail view switch.** [ASSUMPTION: cut, per the brief]
- **Ranking by "recent usage".** Not possible: browsers cannot see when a file was opened, and Find stores nothing.
- **Firefox, Safari, and mobile.**
- **More than one selected folder per session.**
- **Conversational or AI search.** Vision.

## 8. Success Metrics

**Primary**
- **SM-1: Finding files by name** *(rewritten)*. A moderated test of 5 users, each doing 5 tasks with half-remembered **name or type** cues on a seeded folder. *Directional signal, not proof.* [ASSUMPTION: targets]
  - **Protocol:** each user does the same tasks in Find and in their operating system's search, in counterbalanced order. The seeded folder is the user's MyFiles folder, placed in a location the operating system's search indexes.
  - **Timing:** the clock starts in the start state and stops when the correct file is located in File Explorer or Finder.
  - **Target:** a median of 45 seconds or less in Find, including choosing the folder, and at least 3 of 5 users at least as fast in Find.
  - Validates FR-1, FR-6, FR-8, FR-9, FR-17, FR-21.
- **SM-2: Partial name search works** *(simplified)*. On a fixture set of queries using part of a name, a different word order, different separators, or different case, the target file is in the result list 100% of the time. Validates FR-9.

**Secondary**
- **SM-4: Name search latency.** 200 ms or less at 10,000 file entries on the reference machine, including while listing runs. Validates FR-8, NFR-4.
- **SM-6: Privacy and integrity hold.** An automated check of a full session, measured after the initial load, counts zero network requests made by Find itself, zero storage writes, and zero changed files. Validates NFR-1 to NFR-3.
- **SM-9: Portfolio quality.** Rule coverage of at least 90% (NFR-8), and a complete BMAD trail: brief, PRD, architecture, stories.

**Counter-metrics (do not optimize)**
- **SM-C1: Result-list noise** *(simplified)*. Loosening matching (for example, matching *any* query word instead of *every* word) would raise SM-2 but flood the result list. On a fixture set of 30 partial-name queries over a seeded folder of 10,000 file entries (including short query words), keep the median result count at 20 or fewer per query. [ASSUMPTION: the 20-result threshold is unvalidated; check it against the SM-2 fixtures and the SM-1 test, and adjust] Counterbalances SM-2.

*Deferred metrics (§11):* SM-3, SM-5, SM-7, SM-8, SM-C2, SM-C3.

## 9. Open Questions

- **OQ-1. Blocked folders.** *Informs FR-1 and FR-2.* The decision is the MyFiles hint. Still to verify in the architecture spike:
  - Can Chrome and Edge open `C:\MyFiles` and `~/MyFiles` on Windows, macOS, and Linux?
  - Are Downloads and Desktop blocked, for users who don't follow the hint?
  - Are Downloads and Desktop themselves refused, while folders inside them open? (Drag-and-drop is out of scope for the MVP.)
  - What is the most reliable way to locate a file in File Explorer and Finder from a copied location?
- **OQ-2. Real folder sizes.** *Informs NFR-4.* How many files does a typical user's work folder hold? Until this is answered, the NFR-4 numbers are fixed design targets.
- **OQ-3. Name rank vs content rank.** *Deferred* with ranking and content search (§11).
- **OQ-4. Typo limits.** *Deferred* with typo tolerance (§11).
- **OQ-5. Daily folder re-selection.** Users must choose the folder again every session. If the SM-1 test shows this is a deal-breaker, reconsider remembering *only the folder permission*, not file data. That would be an explicit exception to NFR-3.
- **OQ-6. Show in folder.** *Deferred* with the folder tree (§11, FR-19).
- **OQ-7. Name.** Check the availability of "Find" before any public release.
- **OQ-8. Brief out of sync.** *Resolved:* the brief was updated to match the reduced scope.
- **OQ-9. Cloud-only files.** *Low priority.* Find reads only names and dates in the MVP. Whether listing a cloud-synced folder makes the operating system download files is checked in the architecture spike.

## 10. Assumptions Index

Defaults that architecture should treat as fixed design targets: NFR-4. All others can be tuned later.

- §2.3 UJ-1 was structured from the user's narrative and accepted without line-by-line confirmation.
- §3 Type: the extension lists for each type.
- FR-1: the MyFiles hint wording and suggested locations (`C:\MyFiles`, `~/MyFiles`).
- FR-12: the empty-query prompt wording.
- NFR-4: the reference machine, scale limits, timings, and the 100 ms freeze limit.
- NFR-6: WCAG 2.1 AA.
- §7.2: the list/thumbnail switch is cut.
- SM-1 targets, and the SM-C1 threshold of 20 results (to validate).

## 11. Deferred Requirements (post-MVP)

These requirements were part of the approved PRD before the scope reduction on 2026-10-25. They are **not** part of the MVP. For traceability, this section quotes their text **verbatim as last approved**: the full approved PRD is git commit `f0f6934`, plus the one later approved addition quoted in §11.5. When a deferred requirement returns, it is re-validated against the MVP requirements above.

### 11.1 Glossary entries as last approved

*Terms that changed or are used only by deferred requirements. The MVP Glossary (§3) governs the MVP.*

- **Folder tree:** The hierarchical view of the selected folder's subfolders and files. It loads progressively.
- **Listing:** Reading the names and details of every file entry in the selected folder. It runs at the start of the session and on rescan (FR-22). Name search covers file entries as listing reaches them.
- **Name date:** A date found in a file entry's name, such as `invoice_2025-12.pdf` → December 2025. It covers a day, a month, or a whole year, depending on what the name contains (FR-18).
- **Date source:** Where a date comes from: the **modified date** or the **name date**.
- **Token:** One piece of a name or a query. Text is split into tokens at spaces, underscores, hyphens, and dots, and at lower-to-upper case changes (`ProjectReport` → `Project`, `Report`). Letters and digits are *not* split apart, so `Q3` is one token. Matching ignores case.
- **Query:** The text the user types into search. It is split into **query words** by the token rules, so `report-final` gives `report` and `final`.
- **Name match:** A file entry where *every* query word matches its name, as an exact token, a token beginning, text anywhere in the name, or within typo tolerance (FR-9, FR-10).
- **Text-layer PDF:** A PDF that contains extractable text. A PDF without a text layer, such as a scanned document, has no searchable content.
- **Searchable content:** Text extracted from a text file or a text-layer PDF.
- **Content match:** A file entry whose searchable content contains *every* query word (FR-14).
- **Content not searchable:** The label on a file entry whose content was not indexed. It always comes with a reason: no text layer, over the size limit, index limit reached, failed to read, or format not supported yet (FR-16).
- **Content-only match:** A content match that is not also a name match.
- **Match kind:** The label explaining why a file entry is in the result list. Each one is a name match kind or **content**. Best first:
  - **exact:** every query word equals a whole token of the name.
  - **partial:** every query word equals a token or is the beginning of one, and at least one is a beginning.
  - **substring:** every query word appears somewhere in the name, and at least one appears only inside a token (`port` in `report.pdf`). Only query words of 3 or more characters can match this way (FR-9).
  - **content:** a content-only match.
  - **typo:** every query word matches, and at least one matches only within typo tolerance.

  A file entry that is both a name match and a content match gets the better of the two kinds.
- **Content index:** The in-memory store of searchable content for the session. It is built during indexing and discarded when the session ends.
- **Indexing:** Building the content index in the background. While listing or indexing runs, the result list is *still updating*.
- **Filter:** A condition that narrows the result list:
  - **type filter:** one or more types.
  - **date filter:** a date range, applied to one or both date sources.
- **Rank:** The order of the result list.
  - With a query: by match kind, best first (exact, partial, substring, content, typo). Ties go to the most recent modified date.
  - With no query: most recent modified date first.
- **Selected file entry:** The file entry the user has chosen in the result list or the folder tree. There is at most one.
- **Preview:** The read-only view, inside Find, of the selected file entry. It appears in the preview panel; EARS-02 uses that name.
- **Start state:** What Find shows when no folder is selected: the **Choose folder** action, the optional **Set up MyFiles** guide, and a short explanation, with no folder tree, result list, or preview.
- **MyFiles folder:** The recommended selected folder. It is a folder named `MyFiles`, in a local folder that isn't cloud-synced, which the user creates and sets as the browser's download location, so downloads land in one place Find can open. Find never creates it, because Find is read-only (NFR-2). Find offers an optional guide (FR-1).
- **File entry:** A file inside the selected folder as Find knows it: name, type, size, location, modified date, and name date if one is found.
- **Blocked folder:** A folder the browser refuses to grant read-only access to, such as a system folder or the home folder. Whether Downloads and Desktop are blocked is unverified (OQ-1).

### 11.2 MVP requirements: text as last approved, before simplification

#### FR-1: Choose a selected folder

The user can choose one folder on their computer as the selected folder. Realizes UJ-1.

**Consequences (testable):**
- Choosing a folder requires an explicit user action: a click on **Choose folder**. Find never opens the folder picker on its own.
- Find requests read-only access. It never requests write access.
- After access is granted, listing starts, the folder tree begins loading, and name search covers file entries as listing reaches them.
- The start state offers a short, skippable **Set up MyFiles** guide in three steps:
  1. Create a folder named `MyFiles` in a local folder that isn't synced by OneDrive, iCloud, Dropbox, or Google Drive, for example `C:\MyFiles` on Windows or `~/MyFiles` on macOS and Linux. The guide explains why: reading cloud-only files makes the computer download them.
  2. In Chrome or Edge settings, set it as the download location.
  3. Choose it with **Choose folder**.

  [ASSUMPTION: the suggested locations; to be verified by OQ-1]
- Each session, the user still chooses the folder. Find cannot open MyFiles automatically, because it keeps no folder permission (NFR-3).

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

#### FR-6: View file details

The user can see each file entry's name, type, size, location, and modified date, plus its name date when one is found.

**Consequences (testable):**
- Size is shown in bytes below 1 KB, then in KB, MB, or GB with one decimal place.
- The modified date is shown with both date and time. The name date is shown at the precision found in the name (day, month, or year).
- The two dates are labelled **Modified** and **Date in name**, so they can never be confused.

#### FR-8: Search file names

The user can enter a query and get a result list of file entries whose names match it, from the selected folder and all its subfolders. Realizes UJ-1 and EARS-01.

**Consequences (testable):**
- **EARS-01 acceptance:** in a folder containing `report.pdf`, `photo.jpg`, and `report-final.docx`, the query `report` returns `report.pdf` and `report-final.docx`, and not `photo.jpg`.
- Matching ignores case: `REPORT` matches `report.pdf`.
- The result list updates as the user types, without pressing Enter.

#### FR-9: Word-by-word and substring matching

A file entry is a name match when every query word matches its name, in any order. A query word matches when it equals a token, is the beginning of a token, appears anywhere in the name, or is within typo tolerance (FR-10).

**Consequences (testable):**
- `project report` matches `Project_Report_Q3.docx`, `project-report.pdf`, and `ProjectReport.txt` (**exact**).
- `q3 report` matches `Project_Report_Q3.docx`, whatever the word order (**exact**).
- `report-final` matches `report-final.docx` (**exact**), because the query splits into `report` and `final`.
- `proj rep` matches `Project_Report_Q3.docx` (**partial**).
- `port` matches `report.pdf`, and `report` matches `annualreport2025.pdf` (**substring**). This satisfies EARS-01's "names contain the search term" for query words of 3 or more characters.
- `project budget` does **not** match `Project_Report_Q3.docx`, because `budget` matches nothing in the name.
- A query word shorter than 3 characters matches only as an exact token or a token beginning, never as a substring. So `re` does not match `are.pdf`.

#### FR-12: Empty, no-match, and filtered-out states

Find always tells the user why the result list is empty or short, and never claims "no match" before the search has finished. Realizes EARS-03 and EARS-04.

**Consequences (testable):**
- **EARS-03 acceptance:** in a folder containing `report.pdf` and `photo.jpg`, after listing completes and indexing completes or is cancelled, the query `xyz123` shows no file entries and the message "No matching files found".
- **EARS-04 acceptance:** with the same folder, while listing or indexing is in progress, the query `xyz123` shows no file entries and the message "No matches yet. Still searching…". It changes to "No matching files found" once both complete.
- If indexing was cancelled (FR-13), listing has completed, and nothing matches, Find shows "No matching files found" with a second line, "Content search was cancelled. Some files weren't searched." This holds whether the cancel happened during or after listing. While listing is still running, EARS-04 applies even if indexing was cancelled. A rescan (FR-22) clears the cancelled state.
- When active filters hide matches, Find shows how many, for example "Filters hide 12 matches", with a **Clear filters** action. This message takes precedence over the EARS-03 and EARS-04 messages, and it applies whether the visible result list is empty or not.
- With an empty query and no active filter, Find shows the folder tree, not a result list.
- With an empty query and any active filter, the result list contains every file entry that passes the filters, in rank order (most recent modified date first).

**Out of Scope:** a query whose words are split between name and content (for example `budget q3`, where `q3` is only in the name and `budget` is only in the content) matches nothing in the MVP.

#### FR-15: Results that are still updating

While listing or indexing runs, the result list says it is incomplete.

**Consequences (testable):**
- While listing or indexing runs, the result list shows a *still updating* indicator. New matches are inserted in rank order without changing the user's scroll position or the selected file entry.
- When both complete, the indicator disappears.

#### FR-21: Open or copy the location of a file

The user can open a file entry in a new browser tab, or copy its location to find it in other tools. Realizes UJ-1 (resolution).

**Consequences (testable):**
- Double-clicking a file entry, or pressing Enter on it, opens a PDF, image, or text file in a new browser tab, read-only. Find itself makes no network request (NFR-1). Whether the browser's PDF viewer does is verified in OQ-1.
- Text files, including HTML and other code files, open as plain text and never run as a page (NFR-5).
- For the type **other**, double-click selects the file entry and shows the hint "Can't open this type in the browser. Use **Copy location** to find it in File Explorer or Finder."
- **Copy location** is available on every file entry and copies its location (for example `MyFiles/Clients/Acme/report.pdf`) to the clipboard, with a short confirmation.

- **NFR-1 Privacy: no network after load.** After the initial page load, Find itself makes zero network requests for the rest of the session. Everything it needs, including the PDF library, is loaded up front. Opening a file in a tab (FR-21) uses a local in-browser link, not the network. Find includes no analytics or telemetry. *Verify:* browser developer tools show no requests after load during a full session.
- **NFR-4 Performance and capacity.** The reference machine is a laptop with 4 cores, 8 GB RAM, and an SSD, running current Chrome. [ASSUMPTION] The reference workload is 1,000 files: 700 text files averaging 20 KB and 300 text-layer PDFs averaging 500 KB. [ASSUMPTION] These are design targets for architecture:
  - For a selected folder of up to 10,000 file entries, name results appear within 200 ms of the last keystroke. [ASSUMPTION]
  - For up to 50,000 file entries, name results appear within 1 second, and Find shows a notice that the folder is large. Folders larger than that are not supported in the MVP, and Find says so. [ASSUMPTION]
  - Indexing of the reference workload completes within 60 seconds. [ASSUMPTION]
  - The content index holds at most 200 MB of extracted text, measured by Find as the total length of stored searchable content. When the next file would exceed that limit, it is skipped and labelled **content not searchable** with the reason "index limit reached" (FR-16). [ASSUMPTION]
- **NFR-5 Safe previews.** Content shown inside Find is inert: it can never run scripts or reach the network. Files opened in a new tab (FR-21) are displayed by the browser's own built-in viewer, which is outside Find's control. Text files still open as plain text (FR-21). Whether opening a PDF in a tab makes any network request is verified in the OQ-1 spike.
- **NFR-6 Accessibility.** Search, the result list, filters, the folder tree, the preview, and the actions in FR-21 are fully usable by keyboard, and meet WCAG 2.1 AA. [ASSUMPTION]
- **NFR-8 Testable rules.** Matching, ranking, filtering, name-date detection, and indexing rules can be unit-tested without rendering the UI, with at least 90% coverage. This is a stated project goal (portfolio).

### 11.3 Deferred requirements

**Former §4.2 Folder Tree and File Details** *(heading as last approved)*

**Description:** The folder tree shows the selected folder's structure, and the user can browse it at any time, with or without a query. Expanding a folder shows its file entries with their details. Large folders load progressively, so the folder tree is usable before listing finishes.

#### FR-5: Browse the folder tree

The user can expand and collapse folders in the folder tree.

**Consequences (testable):**
- On the reference machine (NFR-4), the first level of the folder tree appears within 1 second of access being granted, for a selected folder of up to 10,000 file entries. [ASSUMPTION]
- Expanding a folder shows its subfolders and file entries. Subfolders are listed before file entries.

#### FR-7: Sort file entries

The user can sort the file entries in an expanded folder of the folder tree, and in the result list, by name, type, size, or modified date, ascending or descending.

**Consequences (testable):**
- In the folder tree, subfolders stay above file entries under every sort.
- In the result list, choosing a sort replaces rank order. The user can return to rank order at any time.

#### FR-10: Typo tolerance (English)

A query word can match a token, or the beginning of a token, despite small English spelling errors. A token beginning is compared at the query word's length, or one letter longer or shorter. So `reprot` is compared with `Report` from `Reports`.

**Consequences (testable):**
- Distance is measured as Damerau-Levenshtein distance (optimal string alignment): inserting, deleting, or substituting one letter, or swapping two adjacent letters, each counts as 1 edit.
- `reprot` matches `report.pdf` (1 edit: a swap). `recipt` matches `receipt.pdf` (1 edit: an insertion). `reprot` matches `Reports.pdf`, against the token beginning `Report`.
- Query words of 4–7 letters tolerate 1 edit. Words of 8 or more letters tolerate 2 edits. Words of 3 letters or fewer get no typo tolerance. [ASSUMPTION]
- Typo tolerance does not apply to substring matching, to digits, or to names in non-Latin scripts.

#### FR-11: Ranked results that explain themselves

The result list appears in rank order (§3 Rank), and every file entry in it shows its match kind.

**Consequences (testable):**
- Each file entry in the result list is labelled **exact**, **partial**, **substring**, **content**, or **typo**.
- `project reprot` returns `Project_Report.pdf` labelled **typo**.
- For the query `report`, `report.pdf` (exact) ranks above `Q3_Reports.pdf` (partial), which ranks above `annualreport2025.pdf` (substring), whatever their modified dates.
- For the query `budget`, `notes.txt` with "budget" in its content (content) ranks above `budgte_2024.xlsx` (typo).

**Former §4.4 Content Search** *(heading as last approved)*

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

#### FR-16: Files whose content is not searched

Find tells the user which file entries were not content-searched, and why.

**Consequences (testable):**
- Each of these is labelled **content not searchable**, with its reason:
  - a PDF without a text layer;
  - a file over the 10 MB per-file limit [ASSUMPTION];
  - a file skipped because the content index reached its limit (NFR-4: "index limit reached");
  - a file that failed to read.
- DOCX and XLSX files are labelled **content not searchable**, with the reason "format not supported yet", so users never assume Word or Excel content was searched.
- A file that fails to read never stops indexing of the other files.
- Find shows which formats content search covers: text files and text-layer PDFs.

**Former §4.5 Filters** *(heading as last approved)*

**Description:** When a query returns too many results, or when the user remembers *what kind* of file it was or *roughly when* they worked on it, filters narrow the result list. Filters work with or without a query (FR-12). Realizes UJ-1 (step 4).

#### FR-18: Smart date filter

The user can limit the result list to file entries whose date falls in a date range, using the modified date, the name date, or both. Realizes UJ-1 (step 4).

**Consequences (testable):**
- The user sets a range (from and to, inclusive), or picks a whole year. Either end of a range may be left empty.
- The user chooses the date source: **Modified or date in name** (default), **Modified only**, or **Date in name only**.
- With the default, a file entry passes when its modified date *or* its name date falls in the range. Example: `invoice_2025-12.pdf`, modified 10 Jan 2026, passes the filter "2025".
- A name date passes when any part of it falls in the range. A year-only name date such as `report_2024.pdf` passes any range that includes part of 2024.
- Each file entry that passes shows which date source matched: **Modified**, **Date in name**, or both, with the matching date.
- Recognized name-date formats are year first: `YYYY-MM-DD`, `YYYY_MM_DD`, `YYYY.MM.DD`, `YYYYMMDD`, `YYYY-MM`, `YYYY_MM`, and a standalone year from 1990 to 2099. Only valid calendar dates count. Formats with the day or month first, such as `03-04-2025`, are not recognized, because they are ambiguous. [ASSUMPTION]
- The date filter combines with the query and the type filter, so all of them must match.

**Former §4.6 Locate, Preview, and Hand Off** *(heading as last approved)*

**Description:** The user confirms they have the right file without leaving Find, then opens it or locates it in their usual tools. Browsers cannot launch desktop apps or reveal full disk paths, so Find opens supported files in a browser tab and copies the location for everything else. Realizes UJ-1 (step 5, climax, and resolution).

#### FR-19: Show a file entry in the folder tree

The user can jump from a file entry in the result list to its place in the folder tree.

**Consequences (testable):**
- Choosing **Show in folder** expands the folder tree to the file entry's folder and highlights the file entry.
- Each file entry in the result list shows its location.

#### FR-20: Preview a file

The user can preview the selected file entry inside Find, read-only. Realizes EARS-02.

**Consequences (testable):**
- **Text files** display as plain text. Previews of files larger than 1 MB are cut off, with a notice. [ASSUMPTION]
- **PDFs** display page by page.
- **Images** display at fit-to-panel size.
- For the type **other**, Find shows the file details and "Preview not available for this file type".
- When the selected file entry is a content match, the preview scrolls to the first matched words. For a PDF, it opens on that page.
- **EARS-02 acceptance:** in a selected folder containing files, with no selected file entry, the preview panel shows "Select a file to preview". In the start state, no preview is shown (§3 Start state).

**Former §4.7 Rescan** *(heading as last approved)*

**Description:** Files keep arriving during the workday. Rescan lets the user refresh what Find knows without choosing the folder again.

#### FR-22: Rescan the selected folder

The user can rescan the selected folder to pick up file entries added, changed, or removed since the last listing.

**Consequences (testable):**
- The folder tree header shows "Listed at HH:MM".
- **Rescan** reruns listing and indexing, and keeps the query, filters, and selected file entry.
- During a rescan, the current folder tree and result list stay visible and usable. File entries are replaced as listing reaches them, and removed file entries disappear when listing completes. The content index is replaced the same way as indexing reaches each file.
- During a rescan, the result list shows *still updating*, and EARS-04 applies.
- If the selected file entry no longer exists, the preview shows "This file is no longer in the folder".
- If the browser no longer grants access, Find returns to the start state with the FR-2 hint.

**Out of Scope:** automatic change detection, and re-indexing only the changed files. A rescan is a full rebuild in the MVP.

### 11.4 Metrics as last approved

- **SM-1: Faster to the right file (supported formats).** A moderated test of 5 users, each doing 5 tasks with half-remembered cues on a seeded folder. *Directional signal, not proof.* [ASSUMPTION: targets]
  - **Task design:** tasks whose cue is a file's content use only text files and text-layer PDFs. Tasks whose cue is a name or a date may use any type. At least 2 of the 5 tasks are solvable only through content or a name date.
  - **Protocol:** each user does the same tasks in Find and in their operating system's search, in counterbalanced order, and the seeded folder is the user's MyFiles folder, placed in a location the operating system's search indexes.
  - **Timing:** the clock starts in the start state, with MyFiles already set up (SM-8), and stops when the correct file is open in a Find tab, or located in File Explorer or Finder.
  - **Target:** a median of 45 seconds or less in Find, including choosing the folder, and at least 3 of 5 users faster in Find.
  - Validates FR-8 to FR-21.
- **SM-2: Forgiving name search works.** On a fixture set of queries with one typo, a partial name, or a fragment from the middle of a name, the target file appears in the top 5 at least 90% of the time. Validates FR-9 to FR-11.
- **SM-3: Content search works.** On a fixture set of queries whose words appear only inside the target text file or text-layer PDF, the target appears in the top 5 at least 90% of the time once indexing completes. The fixture set includes queries where other files' names contain the query words, so content matches compete with substring matches. Validates FR-14, FR-11.
- **SM-5: Indexing throughput.** The reference workload is indexed within 60 seconds, with no freeze over 100 ms. Validates FR-13, NFR-4.
- **SM-7: Name dates are detected correctly.** On a fixture set of file names, every recognized format yields the correct name date, and day-first or month-first names yield none. Validates FR-18.
- **SM-8: MyFiles works.** Measured as a separate task before SM-1: each user follows the MyFiles guide without help. Target: at least 4 of 5 users succeed, and the chosen folder opens on the first try. [ASSUMPTION] Validates FR-1, FR-2.
- **SM-C1: Wrong files in the top 5.** Loosening typo or substring matching raises SM-2 but floods the result list with wrong files. Keep the false-match rate in the top 5 on the SM-2 fixture set at 20% or less. [ASSUMPTION] Counterbalances SM-2.
- **SM-C2: Silently skipped files.** Never speed up indexing by quietly skipping files. Every file entry left unindexed must be labelled (FR-16). Counterbalances SM-5.
- **SM-C3: Over-eager name dates.** Detecting more number patterns as dates would make more files pass date filters, but wrongly. A project code like `ticket_20251` must not become a date. Counterbalances SM-7.

### 11.5 Scope notes and open questions as last approved

Approved after commit `f0f6934` (architecture review, 2026-10-25), and not in git at the time of the reduction:

- **Full-fidelity PDF preview.** To keep zero network requests after load (NFR-1), Find bundles no optional PDF font, character-map, or image-decoder assets.
  - PDFs that rely on non-embedded standard fonts preview with substitute fonts.
  - Text needing character maps (mostly CJK) may extract incompletely.
  - Images that need extra decoders (for example JPEG 2000) or ICC colour profiles may not render in preview.

  Ordinary text-layer PDFs are unaffected. See architecture AD-14 and AD-21.

From commit `f0f6934`:

- **Queries split between name and content** (FR-12).
- **Typo tolerance in content search, and typo tolerance for non-English text.**
- **Name dates written day-first or month-first**, such as `03-04-2025`, because they are ambiguous.
- **Detecting cloud-only files.** If a user selects a synced folder, Find can't tell, and the operating system may download files during indexing (OQ-9).
- **Automatic change detection.** The user rescans instead (FR-22).
- **OQ-3. Name rank vs content rank.** *Decided:* content ranks above typo matches and below substring matches (§3 Rank). Validate the order in the SM-1 test.
- **OQ-4. Typo limits.** Are the FR-10 edit limits right? Tune them against SM-2 and SM-C1.
- **OQ-6. Show in folder.** *Resolved:* FR-19 confirmed in the MVP.
- **OQ-9. Cloud-only files.** *Informs FR-13.* Can Find detect cloud placeholder files from what the browser exposes, so it can skip them? The architecture spike answers this. Until then, the MyFiles guide steers users away from synced folders.
