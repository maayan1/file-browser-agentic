---
title: 'Reconcile inputs: Find architecture spine'
reviewed: ARCHITECTURE-SPINE.md (status: draft, updated 2026-10-25)
inputs: [prd.md (final, filename-only MVP), addendum.md, docs/product/earsRequirments, AGENTS.md]
date: 2026-10-25
---

# Reconcile inputs: Find architecture spine

Scope: every MVP FR/NFR, EARS-01/03/04, the quiet requirements, the AGENTS.md policies, and anything in the spine still shaped by a deferred feature. The spine was not edited.

**Counts:** High 2 · Medium 9 · Low 9 (20 findings)

## Findings

### F1 — High — The 50,000 limit has no notice, and a capped listing could claim "no match"
- **Input:** PRD NFR-4 ("for up to 50,000 file entries … Find shows a notice that the folder is large. Folders larger than that are not supported … and Find says so"); Principle 1 ("never claims 'no match'" before listing has finished); EARS-03 rev. 3.
- **Spine location:** AD-11 (status record `listing: idle | running | done`), AD-13 (`folderTooLarge` session error), Consistency Conventions ("Tunable limits").
- **Gap:** There is no large-folder notice in the snapshot or the message keys. The spine doesn't say what happens at the limit: does listing stop, are the entries already listed kept and searchable, and what is `listing` set to? If listing stops and is marked `done`, AD-11 returns `noMatchFinal` ("No matching files found") even though some files were never listed, which breaks EARS-03's intent and Principle 1. If `folderTooLarge` is a session-level `error`, it may wipe the results the user could still use.
- **Suggested fix:** Add a rule:
  - At 50,000 entries, listing stops. The listed entries stay searchable, and the status becomes a distinct `truncated` value, not `done`.
  - AD-11 never returns `noMatchFinal` when the status is `truncated`. It returns a "not all files were listed" key instead.
  - Add a `largeFolder` flag (or notice key) to the snapshot.
  - Make `folderTooLarge` a notice, not a session error, unless the PM decides otherwise.
  - Ask the PM for the notice wording and for the threshold of the "folder is large" notice, since NFR-4 doesn't say whether it starts at 10,000 entries or only at the limit.

### F2 — High — A subfolder that can't be read is skipped silently
- **Input:** PRD Principle 1 ("Find never silently removes a file entry"), Principle 4 (transparent results), EARS-03.
- **Spine location:** AD-13 ("A failure to read a subfolder is recorded once and listing continues").
- **Gap:** The spine records the failure but never says where it is recorded or shows it to the user. The snapshot `{ status, results, filtersHidden, message, error }` has no field for it. Every file in that subfolder disappears without notice, and AD-11 can still return `noMatchFinal` for a file that sits in it.
- **Suggested fix:** Count the unreadable subfolders in the status record and publish the count, for example as `skippedFolders` in the snapshot. Add a notice key, so the UI shows something like "Some folders couldn't be read" next to `noMatchFinal` and the results. Raise a PRD question for the wording, because FR-6's "unavailable" covers only an entry's details, not whole subfolders.

### F3 — Medium — The status and message model doesn't cover the start state, the FR-2 hint, or unsupported browsers
- **Input:** PRD Glossary "Start state" (Choose folder, the MyFiles hint, and an explanation); FR-1 MyFiles hint; FR-2 (a hint not styled as an error that "stays until the user chooses a folder successfully"); FR-4.
- **Spine location:** AD-9 (snapshot shape), AD-11 (`primary` keys `prompt | results | stillSearching | noMatchFinal`), AD-13 (cancellation "returns to the start state with the FR-2 hint").
- **Gap:** `listing: idle | running | done` can't tell "no folder yet" from "picker closed without access". No key or flag carries the FR-2 hint or keeps it until a folder is chosen successfully. It's also undefined what a cancelled picker does when a folder is already selected: FR-2 says it returns to the start state, which would end the current session.
- **Suggested fix:** Add a session phase to the status record, for example `start | folderNotOpened | unsupported | active`, and expose it in the snapshot. State that `folderNotOpened` lasts until a successful `selectFolder`. Decide, and confirm with the PM, whether a cancelled picker during an active session keeps the current session or ends it. Keep the MyFiles hint text with the S4 fallback.

### F4 — Medium — No port for opening the picker or detecting support, and user activation isn't protected
- **Input:** PRD FR-1 ("Find never opens the folder picker on its own"; it opens only on a click on **Choose folder**), FR-4 (feature detection), addendum (feature-detect for FR-4; do not use `startIn: "documents"`).
- **Spine location:** AD-4 (FileSource "given the granted directory, yields file entries"), AD-9 (`selectFolder`), AD-15 ("Only the FileSource adapter calls the picker").
- **Gap:**
  - No port operation opens the picker or reports whether folder access is supported, so it's unclear how `selectFolder` reaches the picker while the core stays pure.
  - `showDirectoryPicker` needs transient user activation. An `await` before the picker call (for example, resetting state first) can lose it.
  - The addendum's `startIn` warning didn't land.
- **Suggested fix:**
  - Add a picker operation to a port (`FileSource.pick(): granted | notOpened`, or a separate `FolderPicker` port) and an `isSupported()` check used at app start.
  - Rule: the picker is called as the first step of a `selectFolder` started by a user click, with no `await` before it, and never from anywhere else.
  - Rule: never pass `startIn: 'documents'`.

### F5 — Medium — NFR-6 (keyboard, WCAG 2.1 AA) is bound but has no rule
- **Input:** PRD NFR-6; FR-21 ("Copy location is usable by keyboard").
- **Spine location:** frontmatter `binds` (NFR-6); AD-12 (virtual rendering); Deferred ("UI component structure … bmad-ux").
- **Gap:** No AD or test convention covers accessibility. AD-12's virtual rendering is the main risk:
  - rows that are recycled lose focus;
  - Copy location in rows that aren't drawn can't be reached by keyboard;
  - screen readers lose the list's size.
  - The EARS messages, the *still updating* indicator, the filter notice, and the Copy location confirmation need live-region announcements.
- **Suggested fix:** Add an AD with the constraints that bmad-ux can't undo:
  - a virtual list that keeps keyboard focus and exposes the list size and position (`aria-setsize`/`aria-posinset` or `aria-rowcount`);
  - one polite live region for the message key, the *still updating* indicator, the filter notice, and the copy confirmation;
  - a Playwright keyboard-only flow, plus an automated accessibility scan as a dev dependency.

### F6 — Medium — Choosing a new folder doesn't reset the query and filters
- **Input:** PRD FR-3 ("Choosing a different folder discards the previous file entries, query, and filters").
- **Spine location:** AD-8 (generation guard discards entries only), AD-12 (the UI holds "the text being typed").
- **Gap:** The spine discards only the entries. The controller's query and type filter, and the text held in the UI, are not reset.
- **Suggested fix:** In AD-8/AD-9, state that a successful `selectFolder` resets the query, the type filter, and `filtersHidden`, and that the UI clears its text field when the snapshot's generation changes.

### F7 — Medium — Inserting new results can move the scroll position
- **Input:** PRD FR-15 ("New matches are inserted in result order without changing the user's scroll position").
- **Spine location:** AD-7 (snapshots about 10 times per second), AD-12 (virtual rendering, the UI holds the scroll position).
- **Gap:** With a virtual list, entries inserted above the viewport shift what the user sees. No rule anchors the view.
- **Suggested fix:** Add to AD-12: the UI anchors on the first visible entry's ID (its location, AD-10) and restores it after each snapshot. Add a Playwright test that scrolls during listing.

### F8 — Medium — The 100 ms freeze limit is only a spike criterion, not a standing rule
- **Input:** PRD NFR-4 ("No interaction freezes for more than 100 ms during listing"), Principle 3.
- **Spine location:** AD-7 (matching, filtering, and ordering are "synchronous in-memory operations"), spike S3.
- **Gap:** The limit is checked once in S3. Nothing in the spine bounds a full synchronous re-match and re-sort of up to 50,000 entries, or the React render that follows, on each keystroke and each snapshot. There is no regression test either.
- **Suggested fix:** Make "no main-thread task over 100 ms during listing" an AD-7 invariant. Merge newly listed entries in order instead of re-sorting everything. Add a CI performance check: Playwright with a 50,000-entry stubbed folder and a Long Tasks observer.

### F9 — Medium — How a file entry gets its type is not specified
- **Input:** PRD Glossary "Type" (the extension lists; SVG, DOCX, and XLSX are *other*), FR-6, FR-17.
- **Spine location:** AD-4 (FileSource yields name, location, modified date), Consistency Conventions.
- **Gap:** `FileEntry` has no type, and no rule says where the type is derived. Edge cases are open: no extension, dotfiles, multiple dots, and uppercase extensions.
- **Suggested fix:** Add a pure core function that derives the type from the last extension, ignoring case, using the Glossary lists exactly, with everything else as *other*. Cover it with table-driven tests.

### F10 — Medium — The 90% coverage target isn't enforced
- **Input:** PRD NFR-8 ("at least 90% coverage"), SM-9; AGENTS.md policy (rules unit-testable without the UI).
- **Spine location:** Consistency Conventions → Tests; AD-20 (CI checks).
- **Gap:** The spine requires unit tests but sets no coverage threshold, and CI doesn't check one.
- **Suggested fix:** Set a Vitest coverage threshold of 90% or more on `src/core/`, enforced in the CI step of AD-20.

### F11 — Medium — Linux is missing from the spike platforms
- **Input:** PRD NFR-7 (Windows, macOS, and Linux); OQ-1 (`~/MyFiles` "on Windows, macOS, and Linux"; also whether Downloads and Desktop are blocked, and whether drag-and-drop gets around the block).
- **Spine location:** Required Technical Spike intro ("on Windows and macOS"), S4.
- **Gap:** Linux isn't tested, and the remaining OQ-1 sub-questions aren't assigned to any spike.
- **Suggested fix:** Add Linux to S3, S4, and S7, or record the risk explicitly. Add the Downloads/Desktop and drag-and-drop checks to S4, or mark them as out of scope in the decision log.

### F12 — Low — `accessLost` and `folderTooLarge` have no PRD text to map to
- **Input:** PRD (no FR defines an access-lost message; NFR-4 says only "Find says so").
- **Spine location:** AD-13 ("The UI maps each code to the PRD's message or flow").
- **Gap:** `accessLost` looks like a leftover from the file hand-off and Rescan design. The MVP never says when it fires, and the PRD has no text for either code.
- **Suggested fix:** Define when `accessLost` fires in the MVP (for example, `NotFoundError` or `NotAllowedError` during listing) or remove it. Ask the PM for the wording of both codes.

### F13 — Low — Leftovers from deferred features
- **Spine location:** AD-4 (Clock port), AD-20 ("Any future worker URL …").
- **Gap:** The Clock port came from `listedAt` (FR-22 Rescan, deferred) and has no stated MVP use. The worker-URL clause applies only to the AD-5 fallback.
- **Suggested fix:** State Clock's MVP use (timing the AD-7 listing slices), or remove it. Tag the worker-URL sentence as applying only to the S3 fallback.

### F14 — Low — The empty-query edge cases aren't encoded
- **Input:** PRD Glossary "Query" (empty pieces are ignored; a query of only separators is an empty query); EARS-03 and EARS-04 apply only to a non-empty search term.
- **Spine location:** Consistency Conventions → Matching; AD-11 precedence.
- **Gap:** The matching convention doesn't drop empty pieces. AD-11 can return `noMatchFinal` or `stillSearching` for an empty query with an active filter that passes nothing, a case EARS doesn't cover.
- **Suggested fix:** Add "empty pieces are ignored; a separators-only query is empty" to the matching convention. Define what AD-11 returns for an empty query with an active filter and zero results.

### F15 — Low — AD-11 `primary` is undefined when filters hide everything
- **Spine location:** AD-11 step 3.
- **Gap:** Step 3 says the filter notice "replaces the EARS messages", but none of the four `primary` values covers that case.
- **Suggested fix:** Add a `filteredOut` key, or state which existing key is returned and that the UI suppresses it.

### F16 — Low — The result-order collator ignores accents as well as case
- **Input:** PRD FR-8 ("alphabetical by name, ignoring case").
- **Spine location:** Consistency Conventions → Result order (`sensitivity: 'base'`).
- **Gap:** `base` sensitivity also treats `é` and `e` as equal, which is more than "ignoring case".
- **Suggested fix:** Use `sensitivity: 'accent'`, or record the choice in the decision log.

### F17 — Low — The English UI and the date-and-time format aren't fixed
- **Input:** PRD NFR-7 ("The interface is in English"); FR-6 (modified date shown with date and time).
- **Spine location:** Consistency Conventions → Dates and times ("Formatting for display happens only in the UI").
- **Gap:** No locale is set for display formatting. `Intl.DateTimeFormat` with the browser's default locale would produce dates that aren't English.
- **Suggested fix:** State that UI strings are English and that dates are formatted with an `en` locale, showing both date and time.

### F18 — Low — Copy location has no failure path
- **Input:** PRD FR-21 (copy "with a short confirmation"); S6 fallback.
- **Spine location:** AD-12 (transient "copied" confirmation), AD-16.
- **Gap:** The spine doesn't say what happens if `navigator.clipboard.writeText` rejects, or which layer calls the clipboard: the UI directly, or through a port.
- **Suggested fix:** State that the UI calls the clipboard (it is already a browser-facing adapter), shows failure feedback when the write fails, and that e2e tests grant clipboard permission and check the confirmation text.

### F19 — Low — The AGENTS.md overview will go stale
- **Input:** AGENTS.md ("the stack will be chosen in the BMAD architecture step"; the Running and verifying TODO).
- **Spine location:** AD-1.
- **Gap:** AD-1 covers adding the build and test commands but not the stale overview sentence. All the other AGENTS.md policies landed: read-only (AD-15), no network (AD-14), no persistence (AD-16), testable rules (AD-3/AD-11), installer-managed paths (AD-1), Glossary terms (Naming), the product name "Find", and memlog through the script.
- **Suggested fix:** When the spine is finalized, refresh the AGENTS.md block with bmad-project-context: the stack summary, the spine path, and later the commands.

### F20 — Low — Persistence through browser-managed state isn't banned
- **Input:** PRD NFR-3 (no folder permission, query, or filter kept); AGENTS.md persistence policy; addendum (no-persistence note).
- **Spine location:** AD-16.
- **Gap:** The storage APIs are banned, but nothing stops the query or filter from being written to the URL or history (`history.pushState`, a hash). The picker's `id` option, which makes the browser remember the last directory, isn't addressed either.
- **Suggested fix:** Add to AD-16: no query or filter in the URL or history, and no picker `id` option.

## Landed without gaps (for the record)

FR-8 and FR-9 matching (EARS-01 acceptance), the FR-12 precedence and the EARS-03 rev. 3 / EARS-04 rev. 1 keys (AD-11), FR-6 "unavailable" details (AD-13), FR-2 treating cancel and blocked the same (AD-13), FR-17 in the core, Copy location by ID (AD-10), NFR-1 (AD-14), NFR-2 (AD-15), NFR-3 storage APIs (AD-16), NFR-5 (AD-17), the NFR-4 200 ms and 1 s targets (S3), OQ-9 (S8), S6 for locating a file from its location, and deferred features kept out (Deferred section; AD-6 and AD-21 retired).
