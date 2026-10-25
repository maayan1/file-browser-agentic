---
title: "Product Brief: Find"
status: complete
created: 2026-10-25
updated: 2026-10-25
---

# Product Brief: Find

> Items tagged **[ASSUMPTION]** were inferred, not confirmed by the user. The PRD (`prds/prd-file-brwsr-2026-10-25/prd.md`) has since confirmed or replaced them. Where the two differ, the PRD wins.

## Executive Summary

Find is a browser-based file explorer for people who look for files all day. They rarely remember exactly what they named a file or where they saved it. Find runs in Chrome or Edge on the folders the user chooses, does all processing locally, and never changes, uploads, or keeps anything.

Native explorers match names literally. Find searches the way people remember:
- It tolerates typos and partial names, and ranks the results.
- It reads inside text files and text-based PDFs.
- It understands dates written in file names.
- It lets the user preview a file to confirm it's the right one.

Find is a portfolio project meant for real users. The goal is a useful tool built with the BMAD method on a clean, framework-independent core, with fully unit-tested search and analysis logic.

## The Problem

Office and knowledge workers receive new files all day: reports, analyses, invoices, and spreadsheets arrive by email, download, and shared drive. The collection grows faster than anyone organizes it. When they need a file again, they rarely remember its exact name or where they saved it. Was it `Q3_report_final.pdf` or `report-q3 (2).pdf`? Was it in Downloads or in the project folder? And was the invoice from December or January?

[ASSUMPTION] Today they get by in three ways:
- Scrolling through folders sorted by date.
- Retrying the operating system's search with different spellings.
- Opening several near-identical files to work out which one is current.

Each search costs minutes, and it happens many times a day. A worse cost is picking the wrong version and sending it on.

Native file explorers do search, but they match names literally. A typo or a misremembered word returns nothing. They treat `report_v2` and `report_final` as unrelated files, and they don't search by the date written in the file name.

## Who This Serves

**Primary: the everyday file-finder.** This is someone who looks for files on their computer throughout the workday. They keep receiving new documents as part of the job, and the pile only grows. They know *something* about the file they need: part of the name (maybe misspelled), a word inside it, or roughly when it arrived. They don't know its exact name or location.

*Success for them:* they type what they remember, and the right file is at the top of the results within seconds. A preview confirms it's the one, without leaving Find.

**Secondary:** the same features also serve these users, but the product is not designed around them:
- Researchers juggling many versions of the same document.
- Developers inspecting folder structures.
- Students and freelancers searching their own paperwork in bursts, such as all of last year's invoices at tax time.

## The Solution

A file browser that runs in Chrome or Edge, works on the folders the user chooses, and finds files from whatever the user remembers.

- **Pick a folder and start right away.** The recommended folder is **MyFiles**: a folder the user creates and sets as the browser's download location, so every download lands in one place Find can open. The folder tree loads progressively, so even large folders are usable immediately.
- **Type what you remember.** Search tolerates typos, partial words, and names split by spaces, underscores, or hyphens. Results are ranked: exact matches first, then partial matches, then typo matches.
- **Name results appear instantly, and content results follow.** The product reads inside text files and text-based PDFs in the background and adds matches as it finds them. It shows when results are still filling in.
- **Narrow by what you know.** Filter by file type and by date. The smart date filter matches a file if *either* its modified date *or* a date in its name falls in the range. So `invoice_2025-12.pdf`, downloaded in January, still counts as 2025. The product shows which date matched.
- **Check before opening.** Preview images, text, and PDFs, see each file's details, and jump from any result to its place in the folder tree.

Everything happens on the user's machine. The product never uploads, changes, moves, or deletes a file, and it keeps nothing after the tab is closed.

## What Makes This Different

**The honest baseline:** Windows Explorer and macOS Finder already offer folder trees, previews, thumbnails, and fast indexed search. They are installed, integrated with the OS, and faster than a browser app can be. This product does not try to beat them on their own ground.

**Where it is different:**
- **It searches the way people remember.** Typo-tolerant, ranked name search, plus dates read from file names. Native search matches names literally.
- **It reads inside files and explains itself.** It searches inside text files and PDFs, labels every result exact, partial, typo, or content, and shows which date matched.
- **Its safety is built in, not a setting.** Read-only access, no uploads, no index left behind, and nothing to install. That suits shared and locked-down work machines.

**Not a moat:** these features can be copied. The closest existing project is a Chromium-only browser file explorer with a similar feature list (see addendum). This product's edge is the combination: forgiving, content-aware search with strict read-only, stateless behaviour, executed well. Version awareness, which neither native explorer offers, is the planned next step.

## Success Criteria

**For users:**
- [ASSUMPTION] On a folder of 10,000 files, name results appear within about 200 ms of typing.
- [ASSUMPTION] A search with one typo, or with only part of the name, still puts the target file in the top 5 results.
- [ASSUMPTION] Content indexing of 1,000 text and PDF files finishes in about a minute, and the interface stays responsive throughout.
- A year filter finds invoices whose date appears only in the file name.
- Dates in file names are detected correctly, and number patterns such as `ticket_20251` are never mistaken for dates. No file is ever hidden.
- There are zero network requests during file processing and zero writes to the user's files. Both can be checked in the browser's developer tools.
- [ASSUMPTION] In a one-week trial, 5 real users report finding files faster than with their operating system's search.

**For the project (portfolio):**
- The core search and analysis logic runs without the UI framework and is unit-tested. [ASSUMPTION] Target coverage is 90% or more.
- A complete BMAD trail exists: brief, PRD, architecture, stories, and implementation.

## Scope

**In the MVP:**
- Folder selection, tree navigation, file details, and sorting.
- Forgiving search with ranking.
- Type filter and smart date filter.
- Preview for images, text, and PDF.
- Locating a search result in the folder tree.
- Content search for text files and text-based PDFs.
- A guided MyFiles setup for the recommended folder.

**Platform:** Chrome and Edge on Windows, macOS, and Linux.

**Out of the MVP:**
- Version clusters and comparing versions. These come first after the MVP.
- OCR for scanned documents.
- Content search in Word and Excel.
- Duplicate detection and cleanup recommendations (Future).
- [ASSUMPTION] A list/thumbnail view switch.
- Firefox and Safari.
- Conversational or AI search (Vision).
- Any file modification.
- Accounts, sync, and saved history.

## Open Risks and Unknowns

- **Blocked folders.** Browsers may refuse to open sensitive folders such as Downloads, Desktop, or the home folder. Those are exactly where this user's new files arrive.
  - *Decision:* the **MyFiles** folder, created by the user and set as the browser's download location. Find guides the setup; it never creates the folder itself.
  - *To verify:* that MyFiles inside Documents opens in Chrome and Edge on all three operating systems, and how Downloads behaves for users who skip the setup.
  - *Gap:* attachments saved by desktop email and chat apps still land elsewhere.
- **Rescanning every session.** Nothing is persisted, so every session re-reads the folder and re-extracts content. No public benchmarks exist for large folders: one report has Chrome stalling at about 17,000 files. Size limits and a progress UX must come out of the PRD and architecture.
- **Reach.** Supporting Chromium only excludes Firefox and Safari users. Neither browser has plans to add the folder-access API.
- **Name.** "Find" is a common word. It is hard to search for online, hard to protect as a trademark, and easy to confuse with Finder, the `find` command, and Ctrl+F. Its availability is unchecked.

## Vision

- **Version clusters and comparison.** First after the MVP: group possible versions of one document using strict markers, never hiding a file, and compare two versions.
- **Conversational search.** An AI adapter turns "last year's invoices from Cohen" into the same structured query the core already understands. On-device models are preferred, to keep the privacy promise.
- **More formats.** Add formats under the rule in the addendum, in this order: PDF, then DOCX, then XLSX. OCR for scanned documents comes later.
- **Understanding the pile.** On-demand duplicate detection and cleanup recommendations. Deletion would be optional and only with explicit user permission.
- **More browsers**, if Firefox and Safari add folder access.

Long term, Find should be the trusted way to find any file from a vague memory, without giving up privacy.

## Product Principles

1. **Never hide the right file.** Smart features may group, rank, or label files, but never hide them automatically. A smart feature that hides the right file is worse than no feature.
2. **Privacy by design.** All processing happens locally in the browser. Files are never uploaded, and nothing is kept after the session ends.
3. **User control.** The product suggests; the user decides. Access is read-only: nothing is ever modified, moved, or deleted.
4. **Search first.** Finding the right file quickly is the main objective. Every feature must serve it.
5. **Performance first.** Heavier analysis, such as reading file contents, never delays file-name search or navigation.
6. **Transparent results.** The product always shows whether a match is exact or estimated, why files were grouped, and whether results are still filling in.
