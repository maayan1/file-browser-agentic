---
title: "Addendum: PRD Find"
created: 2026-10-25
updated: 2026-10-25
---

# Addendum: PRD Find

This addendum holds technical detail and rejected alternatives that belong to architecture, not the PRD.

## Likely implementation notes (for architecture to confirm)

> **Scope reduced (2026-10-25):** the notes on open-in-tab, typo distance, the index limit, Rescan, Web Worker indexing, PDF, fuzzy matching, safe previews, and name-date detection now apply to **deferred** requirements (PRD §11). They are kept for when those features return. The folder-access, MyFiles, and no-persistence notes still apply to the MVP.

- **Folder access:**
  - File System Access API, `showDirectoryPicker({ mode: "read" })`. Chromium only.
  - Feature-detect it for FR-4.
  - The picker throws `AbortError` both when the user cancels and when the folder is blocked. FR-2 therefore treats both the same way (see Rejected alternatives).
- **MyFiles setup:** the OS picker has its own **New folder** button, so the user can create `MyFiles` inside the picker; the OS creates the folder, not Find (NFR-2 holds). Don't default the picker to Documents (`startIn: "documents"`), because Documents is often cloud-synced (OQ-9). Verify `C:\MyFiles` and `~/MyFiles` on all three operating systems (OQ-1).
- **Open in tab (FR-21):** use `URL.createObjectURL` on the `File`, with no network involved. For text files, wrap the bytes in a new `Blob` with type `text/plain` so HTML never renders (NFR-5). Revoke the URLs when the session ends.
- **Typo distance (FR-10):** Damerau-Levenshtein, optimal string alignment variant, with an early exit once the edit limit is exceeded. Compare a query word against whole tokens and against token prefixes of a similar length.
- **Index limit (NFR-4):** count characters of extracted text (or UTF-16 length × 2 bytes) as the measure. Don't use `performance.memory`, which is non-standard and unreliable.
- **Rescan (FR-22):** a full rebuild in the MVP. `FileSystemObserver` is experimental and out of scope.
- **Off the main thread:** run indexing and content extraction in Web Workers (FR-13, NFR-4).
- **PDF:** a bundled local PDF library (for example, pdf.js) for text extraction and page rendering. Load it from the app bundle, never from a CDN at runtime (NFR-1).
- **Fuzzy matching:** a small, testable edit-distance implementation in the framework-independent core is preferable to a heavy search library. Either way, the FR-10 thresholds must be configurable for tuning.
- **No-persistence check:** do not store `FileSystemDirectoryHandle` in IndexedDB (NFR-3). This is the main thing comparable apps do that Find must not.
- **Safe previews:** render text as text nodes, never as HTML. SVG is excluded from image preview in the MVP because it can carry scripts (NFR-5).
- **Name-date detection:** keep the detector in the testable rule layer (NFR-8). Use a table-driven fixture set for SM-7 and SM-C3. Accept only valid calendar dates; `2025-13-45` is not a date.

## Research carried from the brief

See `briefs/brief-file-brwsr-2026-10-25/addendum.md`. It covers browser support, the blocked-folder list, the large-folder anecdote (~17k files stalling), comparables, and the native baseline.

## Rejected alternatives

- **Ranking by recent usage:** browsers expose no last-opened time, and storing a usage history would break NFR-3.
- **Opening files in their native apps:** browsers can't launch local apps on local files. In-app preview was chosen instead (UJ-1).
- **Modified date only for the date filter:** briefly the default, then reversed by the user. The smart date filter (modified date or name date, with the date source shown) is in the MVP because the modified date often isn't the document's date. Example: an invoice downloaded weeks later.
- **Distinguishing a cancelled picker from a blocked folder:** not possible, because both raise `AbortError`. FR-2 handles them the same way.
