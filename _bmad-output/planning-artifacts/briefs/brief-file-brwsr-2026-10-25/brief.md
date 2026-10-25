---
title: "Product Brief: Find"
status: complete
created: 2026-10-25
updated: 2026-10-25
---

# Product Brief: Find

> Items tagged **[ASSUMPTION]** were inferred, not confirmed by the user. The PRD (`prds/prd-file-brwsr-2026-10-25/prd.md`) has since confirmed or replaced them. Where the two differ, the PRD wins.
>
> **Scope reduced (2026-10-25):** the MVP is now a focused, **filename-only** local file finder. Content search, smart date filtering, previews, the folder tree, ranking, and typo tolerance moved to the Vision. The reasons are recorded in the decision log.

## Executive Summary

Find is a browser-based file finder for people who look for files all day. They rarely remember exactly what they named a file or where they saved it. Find runs in Chrome or Edge on the folder the user chooses, does all processing locally, and never changes, uploads, or keeps anything.

The MVP does one thing well: it **finds files by name**.
- It matches any part of a file name, ignoring case and word order.
- It lists results clearly: name, type, location, and modified date.
- It narrows by file type.
- It copies a file's location so the user can open the file in their usual tools.

Searching inside files, smart date filtering, previews, and typo tolerance come after the MVP.

Find is a portfolio project meant for real users. The goal is a useful tool built with the BMAD method on a clean, framework-independent core, with fully unit-tested search logic.

## The Problem

Office and knowledge workers receive new files all day: reports, analyses, invoices, and spreadsheets arrive by email, download, and shared drive. The collection grows faster than anyone organizes it. When they need a file again, they rarely remember its exact name or where they saved it. Was it `Q3_report_final.pdf` or `report-q3 (2).pdf`? Was it in Downloads or in the project folder? And was the invoice from December or January?

[ASSUMPTION] Today they get by in three ways:
- Scrolling through folders sorted by date.
- Retrying the operating system's search with different spellings.
- Opening several near-identical files to work out which one is current.

Each search costs minutes, and it happens many times a day. A worse cost is picking the wrong version and sending it on.

Native file explorers do search, but they match names literally. A typo or a misremembered word returns nothing. They treat `report_v2` and `report_final` as unrelated files, and they don't search by the date written in the file name.

## Who This Serves

**Primary: the everyday file-finder.** This is someone who looks for files on their computer throughout the workday. They keep receiving new documents as part of the job, and the pile only grows. They know *something* about the file they need, such as part of its name or its type. They don't know its exact name or location.

*Success for them:* they type the part of the name they remember, and the file appears in a clear list within moments. They copy its location and open it in their usual app.

**Secondary:** the same features also serve these users, but the product is not designed around them:
- Researchers juggling many versions of the same document.
- Developers inspecting folder structures.
- Students and freelancers searching their own paperwork in bursts, such as all of last year's invoices at tax time.

## The Solution

A file finder that runs in Chrome or Edge, works on the folder the user chooses, and finds files from whatever part of the name the user remembers.

- **Pick a folder and start right away.** Find asks for read-only access. A short hint suggests a dedicated **MyFiles** folder for downloads, because browsers refuse to open some system folders.
- **Type any part of the name.** Each word typed must appear somewhere in the file name, in any order and ignoring case. So `report q3` finds `Q3_Report_final.pdf`.
- **See clear results.** They are sorted alphabetically by name, then by location. Each one shows name, type, location, and modified date.
- **Narrow by type.** Filter by text file, PDF, image, or other. If the filter hides matches, Find says how many.
- **Hand off.** Copy a file's location to find and open it in File Explorer or Finder.

Everything happens on the user's machine. The product never uploads, changes, moves, or deletes a file, and it keeps nothing after the tab is closed.

## What Makes This Different

**The honest baseline:** Windows Explorer and macOS Finder already offer folder trees, previews, thumbnails, and fast indexed search. They are installed, integrated with the OS, and faster than a browser app can be. This product does not try to beat them on their own ground.

**Where it is different (MVP):**
- **It matches the way people remember names.** Any part of a name, any word order, with separators ignored. A forgotten prefix or a different word order still finds the file.
- **Its safety is built in, not a setting.** Read-only access, no uploads, nothing left behind, and nothing to install. That suits shared and locked-down work machines.
- **It is the same everywhere.** One tool on Windows, macOS, and Linux, in the browser.

**Not a moat:** these features can be copied, and the MVP's advantage over native search is modest by design. It is the foundation that typo tolerance, content search, and smart dates (the Vision) build on.

## Success Criteria

**For users:**
- [ASSUMPTION] On a folder of 10,000 files, name results appear within about 200 ms of typing.
- [ASSUMPTION] A search with only part of the name, or the words in a different order, finds the target file.
- There are zero network requests after the app loads and zero writes to the user's files. Both can be checked in the browser's developer tools.
- [ASSUMPTION] In a one-week trial, 5 real users report finding files by name at least as fast as with their operating system's search.

**For the project (portfolio):**
- The core search logic runs without the UI framework and is unit-tested. [ASSUMPTION] Target coverage is 90% or more.
- A complete BMAD trail exists: brief, PRD, architecture, stories, and implementation.

## Scope

**In the MVP:**
- Folder selection with read-only permission, and a short MyFiles hint.
- File-name search across the selected folder and its subfolders: case-insensitive, any part of the name, any word order.
- A clear results list (name, type, location, modified date), sorted alphabetically.
- Type filter, with a notice when it hides matches.
- Copy location.
- Clear empty states and basic error handling.
- Local-only processing, no file modifications, and no persistent storage.

**Platform:** Chrome and Edge on Windows, macOS, and Linux.

**Out of the MVP (moved to the Vision unless noted):**
- Content search inside files and PDFs, with indexing and its progress states.
- Previews of any kind, and opening files in a browser tab.
- Smart date filtering and dates read from file names.
- The folder tree and Show in folder.
- Ranking, match labels, and user-selectable sorting.
- Typo-tolerant search.
- Rescan (the user chooses the folder again instead).
- Version clusters and comparing versions.
- OCR, and content search in Word and Excel.
- Duplicate detection and cleanup recommendations.
- [ASSUMPTION] A list/thumbnail view switch.
- Firefox and Safari.
- Conversational or AI search.
- Any file modification.
- Accounts, sync, and saved history.

## Open Risks and Unknowns

- **Blocked folders.** Browsers refuse to open some sensitive folders themselves, such as Downloads, Desktop, Documents, or the home folder. Folders inside them are allowed. Those top-level folders are exactly where this user's new files arrive.
  - *Decision:* a short hint recommends a **MyFiles** folder, created by the user and set as the browser's download location. Find never creates the folder itself.
  - *To verify:* that a local, non-synced MyFiles folder (`C:\MyFiles`, `~/MyFiles`) opens in Chrome and Edge on all three operating systems.
  - *Gap:* attachments saved by desktop email and chat apps still land elsewhere.
- **Listing every session.** Nothing is persisted, so every session lists the folder again. No public benchmarks exist for large folders: one report has Chrome stalling at about 17,000 files. The architecture spike checks responsiveness.
- **Modest advantage.** Without typo tolerance or content search, the MVP mainly adds word-order-free partial matching and safety over native search. The Vision features carry the bigger advantage.
- **Reach.** Supporting Chromium only excludes Firefox and Safari users. Neither browser has plans to add the folder-access API.
- **Name.** "Find" is a common word. It is hard to search for online, hard to protect as a trademark, and easy to confuse with Finder, the `find` command, and Ctrl+F. Its availability is unchecked.

## Vision

- **Smarter name search.** Typo tolerance and ranked results with match labels.
- **Content search.** Search inside text files and PDFs, with background indexing, then DOCX, then XLSX, then OCR, under the format rule in the addendum.
- **Smart dates and previews.** Dates read from file names, a date filter, previews of text, images, and PDFs, opening files in a tab, the folder tree, Show in folder, and Rescan.
- **Version clusters and comparison.** Group possible versions of one document using strict markers, never hiding a file, and compare two versions.
- **Conversational search.** An AI adapter turns "last year's invoices from Cohen" into the same structured query the core already understands. On-device models are preferred, to keep the privacy promise.
- **Understanding the pile.** On-demand duplicate detection and cleanup recommendations. Deletion would be optional and only with explicit user permission.
- **More browsers**, if Firefox and Safari add folder access.

Long term, Find should be the trusted way to find any file from a vague memory, without giving up privacy.

## Product Principles

1. **Never hide the right file.** Filters and other features may narrow results, but never hide files silently. When a filter hides matches, Find says so.
2. **Privacy by design.** All processing happens locally in the browser. Files are never uploaded, and nothing is kept after the session ends.
3. **User control.** The product suggests; the user decides. Access is read-only: nothing is ever modified, moved, or deleted.
4. **Search first.** Finding the right file quickly is the main objective. Every feature must serve it.
5. **Performance first.** Listing a large folder never freezes search or the interface.
6. **Transparent results.** The product always says whether results are still filling in, and why the list is empty.
