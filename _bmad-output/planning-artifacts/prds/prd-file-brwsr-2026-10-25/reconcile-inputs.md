---
title: "Reconcile: PRD Find vs. inputs"
created: 2026-10-25
---

# Reconcile: PRD Find vs. inputs

**PRD checked:** `prd.md`, `addendum.md`, `.memlog.md` (this folder).
**Inputs checked:** brief `brief.md`, `addendum.md`, `.memlog.md`; `docs/product/earsRequirments`; `docs/product/file-browser-idea.md`. `docs/product/smsrtlocalbrowser` (the user's original prompt, cited by the brief memlog) was also read for its constraints.

**Not reported (deliberate departures in the PRD memlog):** version clusters and comparison out of the MVP, smart date filter out, date filter on modified date only, name "Find". Gaps 1 and 5 below refer to the date filter only in how it is *shown* to users, not to its scope.

## 1. EARS traceability

| EARS | Requirement | PRD acceptance | Status |
|---|---|---|---|
| EARS-01 | When a search term is entered, show all files in the selected folder and subfolders whose names contain it | FR-8, "EARS-01 acceptance" (same fixture, same expected result) | Mapped. Note: FR-9/FR-10 widen matching beyond "contains", so on a richer fixture (for example a `repot.txt`), the "only these files appear" wording of EARS-01 no longer holds. FR-8 should say that EARS-01 is the *exact* subset and that forgiving matches are additional, labelled results. |
| EARS-02 | While no file is selected, the preview panel shows "Select a file to preview" | FR-20, "EARS-02 acceptance" | Mapped. Unspecified: whether selection is cleared, and the message shown again, when the query or filters change and the selected entry leaves the result list. |
| EARS-03 | If no files match a non-empty term, show "No matching files found" in the results area | FR-12, "EARS-03 acceptance"; also cited in the UJ-1 edge cases | Mapped, but it interacts with FR-15 and the filters in ways the PRD leaves open (see Gap 2). |

EARS project constraints:

| Constraint | PRD | Status |
|---|---|---|
| All processing local in the browser | NFR-1, §1, §6 | Covered |
| No network requests or server uploads | NFR-1 | **Narrowed** (Gap 3) |
| Read-only access to user-selected folders | FR-1, NFR-2 | Covered |
| No persistent storage of contents or metadata | FR-3, NFR-3 | Covered (OQ5 flags a possible future exception, explicitly) |
| User must explicitly select the folder | FR-1 | Covered |

## 2. Gaps

### Gap 1: Product principles and qualitative intent dropped (High)

The brief's six **Product Principles** (never hide the right file, privacy by design, user control, search first, performance first, transparent results) are not in the PRD as principles. The idea doc's "Simple User Experience: intuitive and easy to navigate" and "simple, fast, intuitive" are also missing, as are the vision doc's five principles (simplicity/no onboarding, speed, focus, privacy, extensibility), which the PRD memlog itself notes (finalize line). The PRD has only the "Core principle" line.

Several are partly encoded (privacy → NFR-1/3, read-only → NFR-2, performance first → FR-13, transparency → FR-11/15/16, never hide → SM-C2). But downstream UX, architecture, and stories lose the *decision rules*:
- **Never hide the right file** governs how strict FR-14 (every query word must appear) and the FR-10 thresholds can be, and how filters signal what they exclude. No requirement states it.
- **Search first** was the brief's test for every feature. The PRD's reason for deferring version clusters ("the MVP proves search first") relies on it without defining it.
- **Simplicity / no onboarding** has no requirement or NFR. Nothing constrains first-run steps, settings, or UI density.

**Fix:** add a short "Product Principles" section after §1 Vision. List the brief's six principles plus simplicity (no onboarding, usable on first visit), each with a one-line pointer to the FRs or NFRs that realize it. Use them as tie-breakers for the open questions in §9.

### Gap 2: EARS-03 is undefined while indexing is running and when filters are active (High)

FR-12 shows "No matching files found" when nothing matches, and FR-15 says results are *still updating* during indexing. The PRD does not say what happens when a query has no name matches while indexing is still running. Showing "No matching files found" then is premature and can be false once content matches arrive. That breaks the brief's transparency principle ("whether results are still filling in") and could lead the user to give up on a file that is there.

The PRD also does not cover the case where the query matches files but the type or date filter excludes all of them. The message is then misleading, and the cause is hidden.

**Fix:** add FR-12 consequences:
- While indexing runs, a no-result state shows the *still updating* indicator with wording like "No matches yet, still searching file contents". The final "No matching files found" appears only after indexing completes or is cancelled.
- When filters remove every match, Find says that filters are hiding N matches and offers a **Clear filters** action.

### Gap 3: The no-network constraint is narrowed (Medium)

The inputs say "No network requests during application use" (`smsrtlocalbrowser`, brief addendum) and "No network requests or server uploads" (EARS constraints). NFR-1 promises only that "processing a selected folder makes zero network requests". That leaves room for requests during use that are not folder processing, such as fonts, update checks, or lazy-loaded chunks. The brief memlog had already flagged the same softening in the NXUSR draft as an unresolved event.

**Fix:** reword NFR-1 to "After the initial page load, Find makes zero network requests for the rest of the session". All code, including the PDF library and fonts, is bundled with the initial load. SM-5 already measures a full session, so the metric then matches the rule.

### Gap 4: The blocked-folder mitigation the user chose was dropped (Medium)

The brief records the user's chosen mitigation for blocked folders: create a dedicated work folder and set it as the browser's download location. It also records a known gap (mail and chat attachments still land elsewhere) and alternatives to verify (subfolders of Downloads, drag-and-drop). The PRD keeps only the verification question (OQ1). FR-2's message gives the reason and offers **Choose folder** again. It gives no guidance, even though Downloads and Desktop are where the primary user's new files arrive.

**Fix:** extend FR-2 so the blocked-folder message suggests a concrete next step: pick a subfolder, or use a dedicated work folder set as the download location. Note the attachment gap in OQ1, and that the FR-2 wording depends on the spike's outcome.

### Gap 5: Transparency items from the NXUSR draft and brief are only partly carried (Low)

The brief addendum lists the NXUSR transparency requirements. Two have no PRD requirement:
- Showing **which formats are supported** for content search. FR-16 labels files that failed, but never says that DOCX or XLSX are not searched by design. A user may assume their Word file was searched and found nothing.
- The **date basis** of the date filter. With modified date only, the UI should say so. A user filtering by "December" otherwise assumes document dates and silently misses the January-downloaded invoice, which conflicts with "never hide the right file" and "transparent results". The *scope* decision is deliberate. The gap is that the UI does not disclose it.

**Fix:** FR-18 adds "the filter is labelled *Modified date*". FR-13/FR-16 add a visible note naming which formats are content-searched, with DOCX and XLSX entries labelled "content not searched (format not supported)" rather than left unlabelled.

### Gap 6: The read-only guarantee is not verified (Low)

The brief success criterion is "zero network requests **and zero writes to the user's files**, both checkable in developer tools". SM-5 checks network and storage only. NFR-2 has no *Verify* line, unlike NFR-1 and NFR-3.

**Fix:** add a Verify line to NFR-2 (for example, the access request uses mode "read", and the folder's checksums or modified dates are unchanged after a full session) and include it in SM-5.

## 3. Checked and consistent (no action)

- Idea doc features: folder selection, tree navigation, name search, metadata, image preview, folders-before-files sorting → FR-1, FR-5, FR-8, FR-6, FR-20, FR-5/FR-7.
- Candidate feature list (brief addendum): every item is covered, except the list/thumbnail switch, which is cut as an [ASSUMPTION] in line with the brief.
- Framework-independent, unit-tested core → NFR-8, SM-6.
- Brief risks: Chromium-only reach → §2.2, FR-4. Rescanning and large-folder limits → NFR-4, OQ2, OQ5. Name → OQ6.
- NXUSR performance items: Web Workers (addendum), visible progress and cancel (FR-13), progressive results (FR-15). In-session index reuse is implied by the session-scoped content index.
- Brief success metrics (200 ms, top-5, 1,000 files in about 1 minute, 5-user trial) → NFR-4, SM-1 to SM-4. The year-filter-by-name metric was dropped consistently with the smart date filter deferral.
- Out-of-scope lists agree: OCR, DOCX/XLSX, modification, accounts and sync, history, conversational AI, duplicates and cleanup.
