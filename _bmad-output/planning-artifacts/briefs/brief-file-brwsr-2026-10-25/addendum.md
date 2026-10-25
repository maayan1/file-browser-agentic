---
title: "Addendum: Smart Local File Browser"
created: 2026-10-25
updated: 2026-10-25
---

# Addendum: Smart Local File Browser

Detail the user contributed that belongs in downstream documents (PRD, architecture) rather than the brief.

## Candidate feature list (as stated by the user, 2026-10-25, before MVP scoping)

1. Select a local folder from their computer.
2. Display folders and files in a hierarchical folder tree.
3. Navigate through folders and subfolders.
4. Search for files by name.
5. Filter files by type.
6. Select a search result and locate it in the folder tree.
7. View file details such as name, size, type, and last modified date.
8. Preview supported files, including images, PDFs, and text files.
9. Switch between list view and thumbnail view.
10. Sort files by name, type, size, or date.

Not yet prioritized. MVP cut is still open.

## Technical constraints (as stated)

- Runs locally in the browser; no backend server.
- No network requests during use; no uploads to external servers.
- Read-only access to user-selected folders.
- No persistent storage of file contents or metadata.
- Core business logic must be framework-independent.

## Existing EARS requirements

`docs/product/earsRequirments` holds three EARS requirements with acceptance tests (search by name, empty-preview state, no-results message). Use them as PRD input.

## Research digest: browser folder access and landscape (2026-10-25, web research)

**Folder access APIs**
- `showDirectoryPicker` (File System Access API) works only in Chromium browsers: Chrome and Edge 86+, and Opera.
  - Firefox's standards position is "negative", and the bug has no target release.
  - Safari/WebKit is sceptical.
  - Chrome for Android support is claimed but unverified.
  - Sources: caniuse.com/mdn-api_window_showdirectorypicker, mozilla/standards-positions#154, bugzilla 1909237, webkit bug 231706.
- The picker needs HTTPS and a user gesture. Read mode is the default, so the user sees a single prompt. Access lasts while the origin's tabs stay open.
- `<input webkitdirectory>` fallback (Baseline since Aug 2025):
  - Returns a flat list of the whole tree at once, so it cannot lazy-load.
  - Large folders stall or exhaust memory.
  - Files carry `lastModified`; folders carry no dates. Empty folders are likely missing (inferred).
- Drag-and-drop with `webkitGetAsEntry` works in all browsers and reads entries in batches, which allows lazy loading. Folders carry no dates. `getAsFileSystemHandle` is Chromium-only and experimental.

**Practical limits**
- No published benchmarks exist for 100k+ files. A 2021 anecdote reports Chrome hanging on a folder of about 17k files (GoogleChromeLabs/browser-fs-access#32).
- Plan for lazy loading one folder at a time, progressive search, and parallel `getFile()` calls.
- The spec lets browsers block sensitive folders: system folders, the whole home folder, the default Downloads folder. The picker then throws `AbortError`. Drag-and-drop bypasses this blocklist.

**Comparables**
- *teodor-andonov PWA file explorer:* nearly the same feature set, but Chromium-only, persists handles in IndexedDB, and edits text.
- *LocalGalleryViewer:* image and video gallery that caches in IndexedDB and has no PDF or text preview.
- *vscode.dev:* Chromium-only for local folders; built for code.
- *MakeQuickWork File Viewer:* views single files only and doesn't browse folders.
- No comparable found is cross-browser, read-only, and stateless together. That combination is the most defensible differentiator.

**Native baseline**
- Windows Explorer has a preview pane, a details pane, thumbnails, sort and group, and indexed search.
- macOS Finder has Gallery view, a Preview pane, Quick Look, and Spotlight.
- Native file managers beat a browser app on speed, thumbnails, and indexed search.
- An honest pitch rests on: no install, guaranteed read-only, the same on every platform, and works on locked-down machines.

## Proposed rule for adding content-search formats (suggested 2026-10-25, pending user confirmation)

Add a file format to content search only when all three hold:
1. **It serves the hero scenario:** real users keep that kind of document in that format.
2. **It extracts fully in the browser:** a bundled library, no network, no server.
3. **It fits the performance budget:** name results stay instant, and content results fill in progressively without freezing the UI.

Proposed order: text (MVP) → PDF with a text layer (no OCR for scanned PDFs) → DOCX → XLSX.
Per-file size cap for content reading: [ASSUMPTION] skip files over ~10 MB in the MVP.

## NXUSR PRD draft (pasted by the user, 2026-10-25)

The user pasted a full PRD draft (sections 1–11: overview, functional requirements 2.1–2.8, UX, privacy, technical, MVP scope table, out of scope, future, principles, success criteria, vision). Only the parts relevant to the brief were pulled in; see `.memlog.md`. Ask the user to supply the full draft as an input when running `bmad-prd`.

PRD-level detail worth carrying forward from it:
- **Transparency requirements:** show which files are scanned or still processing, which formats are supported, why a file was grouped or flagged, whether a match is exact or estimated, and whether results are complete.
- **Performance requirements:** progressive loading, background work in Web Workers, visible progress, cancellable long operations, and in-session index reuse.
- **Architecture modules:** UI, file-system access, search engine, content extraction, date filtering, version detection, duplicate detection, comparison, recommendations. Core logic is framework-independent and unit-tested.
- **Out of scope:** OCR, DOCX/XLSX content, deleting, modifying or renaming files, cloud sync, accounts, persistent history, server processing.
- **Future:** OCR, DOCX/XLSX, deeper similarity, better version detection, optional deletion with explicit permission, more browsers.
