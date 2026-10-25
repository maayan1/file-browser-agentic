# PRD Quality Review — PRD: Find

- **PRD:** `_bmad-output/planning-artifacts/prds/prd-file-brwsr-2026-10-25/prd.md` (+ `addendum.md`)
- **Rubric:** `.claude/skills/bmad-prd/assets/prd-validation-checklist.md`
- **Calibration:** portfolio project with real users; browser-only, read-only, stateless; focused PRD of about 5–8 pages. Feeds UX, architecture, and stories, so downstream usability counts.

## Overall verdict

This is a disciplined, well-sized PRD. It has a clear thesis (forgiving, content-aware search beats OS search), honest scope cuts, product-specific NFRs, and FRs that mostly have concrete, example-driven consequences. Two things put the build at risk. First, FR-2 requires Find to tell a blocked folder apart from a cancelled picker, which the addendum says the platform cannot do, and the unresolved blocked-folder question (OQ-1) concerns exactly the folders where the primary user's files land. Second, the name-match kinds that drive ranking, labelling, and SM-2 are under-defined and contradict each other: "partial" means "some words", the separator list omits camelCase, and mixed queries such as UJ-1's own `project reprot` have no defined match kind. Smaller issues are a handful of Glossary synonyms, an undefined empty-query-plus-type-filter state, and unbounded performance and memory conditions. All of them can be fixed in one Update pass.

## Decision-readiness — strong

Decisions are stated as decisions. The read-only stance (§1, NFR-2), no persistence (NFR-3), and modified-date-only filtering are each paired with what they cost. §7.2 names the known gap ("an invoice downloaded in January won't appear under December"). OQ-5 frames the folder re-selection pain as a possible, explicit exception to NFR-3 rather than hiding it. The single `[NOTE FOR PM]` (§7.2, version clusters) sits at a real tension: the brief's strongest differentiator was deferred. The Open Questions are genuinely open, and each has an owner or a validation path (OQ-3 and OQ-4 point to SM-1, SM-2, and SM-C1).

### Findings
- **low** OQ-1 has an owner but no deadline (§9.1). "*Owner:* architecture spike" doesn't say the spike must finish before UX designs the entry and error flow. *Fix:* add "Resolve before UX finalizes FR-2 flows" (see the Scope honesty finding on phase-blockers).

## Substance over theater — strong

There is one persona (Daniel), and he drives real decisions: progressive loading, the date filter, preview-to-confirm, and scanned-PDF labelling. The NFRs carry thresholds (200 ms, 60 s per 1,000 files, 100 ms freeze, 10 MB limit) and verification methods (NFR-1, NFR-3). The Vision can't be swapped into another PRD: its "MVP has one job" paragraph is a falsifiable bet. NFR-8 is a project goal rather than a user need, and it says so ("This is a stated project goal (portfolio)"), which is honest rather than theater.

### Findings
- **low** NFR-8 prescribes architecture (§5 NFR-8). "Live in a core that is independent of the UI framework" is an architecture decision placed among the product NFRs. It is defensible as a portfolio constraint, but it is the clearest implementation leak in the main body. *Fix:* rephrase as an outcome ("Matching, ranking, filtering, and indexing rules are unit-testable without rendering the UI, with ≥90% coverage"), and let architecture choose the structure.

## Strategic coherence — strong

The thesis is explicit (§1, ¶3), and the feature order follows from it: name search, then content search, then filters, then preview to confirm. SM-1 tests the thesis directly against the OS-search baseline. SM-C1 and SM-C2 are well-chosen counter-metrics tied to specific primary and secondary SMs. The MVP is problem-solving in kind, and the scope logic matches: version clusters were cut *because* the MVP has to prove search first.

### Findings
- **medium** Nothing isolates the "content-aware" half of the thesis (§8). The thesis is "forgiving, content-aware search", but SM-2 covers only name queries (typo or partial). SM-1 lumps all of FR-8 to FR-20 together ("Validates FR-8 to FR-20"), so a passing SM-1 could come entirely from name search. *Fix:* add to SM-2 (or a new SM) a fixture subset of phrase-only queries whose target appears in the top 5 at a stated rate. Alternatively, have SM-1 tag which tasks are solvable only through content.
- **low** SM-1's baseline protocol is unspecified (§8 SM-1). "Faster than with their operating system's search" doesn't say whether the same users do the same tasks in both tools, in what order, or on which OS. *Fix:* one sentence: within-subject, counterbalanced order, and native search on the participant's own OS over the same seeded folder.

## Done-ness clarity — adequate

Most FRs are strong. The EARS-anchored acceptance examples (FR-8, FR-12, FR-20), the concrete matching examples (FR-9, FR-10), and the exact indicator behavior (FR-15) give story writers real acceptance criteria. The weak spots are concentrated in the definitions that the core feature relies on, and in a few unbounded conditions.

### Findings
- **high** The name-match kinds are under-defined and contradict each other (§3 Name match, FR-9, FR-10, FR-11, UJ-1 step 2).
  - The Glossary defines **partial** as "*some* of the query's words match". Taken literally, `project report` matches every file containing "project", which contradicts the precise intent of FR-9 and the AND semantics of FR-14 ("every word of the query").
  - The Glossary's separator list ("spaces, underscores, and hyphens") omits camelCase and no-separator names, yet FR-9's acceptance example requires `ProjectReport.txt` to match. FR-9 also requires `Project_Report_Q3` to match `q3 report`, which implies letter-to-digit boundaries that nothing defines.
  - No rule assigns a match kind to a mixed query. UJ-1's headline query `project reprot` has one exact word and one typo word, and it is a multi-word query that doesn't appear as a substring. Is it partial, typo, or neither? FR-11's labels and rank order can't be tested for it.
  - Ranking, labelling, SM-2, and SM-C1 all depend on these definitions.

  *Fix:* define name matching per query word. A file entry is a name match when **every** query word matches a name token, as exact, prefix, or typo (within FR-10 limits). Tokens split on spaces, `_`, `-`, `.`, camelCase, and letter/digit boundaries. The match kind is the weakest kind across the query words, and "exact" stays as "whole query is a case-insensitive substring". Add `project reprot → Project_Report.pdf, labelled typo` as an acceptance example.
- **high** FR-2 can't be built or tested as written (FR-2; addendum "Folder access"). FR-2 needs one outcome for a blocked folder (a message naming the reason) and another for a cancel (start state, no error). The addendum states that "the picker throws `AbortError` both when the user cancels and when the folder is blocked". So the two consequences can't both be verified, and the PRD doesn't say which one wins. *Fix:* make FR-2 outcome-based and robust to the ambiguity. For example: "When the picker closes without granting access, Find returns to the start state and shows a non-error hint that some system and personal folders can't be chosen, with **Choose folder** again." Make OQ-1 (see Scope honesty) a gate before UX.
- **medium** The empty-query-plus-type-filter state is undefined, and rank with no query is undefined (FR-12, FR-17, FR-18, §4.5). §4.5 says "Filters work with or without a query". FR-18 defines empty query plus date filter ("every file entry in the range, newest first"), and FR-12 defines empty query with no filter (folder tree). Empty query plus a type filter alone has no defined result or order, and Rank (§3) is defined only relative to a query. *Fix:* add one consequence: "With an empty query and any active filter, the result list contains every file entry that passes the filters, newest first." Move the "newest first" rule into the Rank glossary entry.
- **medium** Performance conditions are unbounded (FR-5, NFR-4). "Mid-range laptop" (NFR-4) isn't defined, and FR-5's 1-second target has no hardware condition at all. "Folders larger than 10,000 file entries still work, more slowly" has no bound, although the brief addendum records ~17k files stalling a comparable app. *Fix:* name a reference machine class (for example 4 cores, 8 GB RAM, SSD) once in NFR-4 and reference it from FR-5. Bound the >10k case ("up to 50,000 file entries, name results within 1 s, with a visible notice") or explicitly declare it unsupported beyond N.
- **medium** No memory bound for the in-memory content index (§3 Content index, FR-16, NFR-4). The 10 MB per-file limit alone allows a multi-GB in-memory index on a large folder, and nothing caps total memory or says what happens when a cap is reached. For an architecture that keeps everything in memory, this is a design input. *Fix:* add an NFR-4 bullet for a total content-index budget (or a cap on indexed files), with the FR-16 behavior when it is exceeded: files beyond the cap are labelled **content not searchable**, with the reason, which satisfies SM-C2.
- **low** Small unbounded terms. FR-14 "a short snippet" (no length). FR-6 "KB, MB" (no rule for GB or bytes). FR-13 "files indexed out of total" (is the total all file entries, or only text files and PDFs?). FR-12's requirement statement "handles … clearly" is an adjective, although its consequences are concrete. *Fix:* give the snippet a character or word bound, say "total = file entries with a text-file or PDF type", and reword FR-12's statement.

## Scope honesty — strong

§6 Non-Goals and §7.2 Out of Scope both do real work: the one-folder limit, no ranking by recent usage, no typo tolerance in content search, and the DOCX/XLSX/OCR ordering. Inline `[ASSUMPTION]` tags are used consistently and indexed. Open-items density is about 7 OQs plus about 15 assumptions, which is acceptable for a portfolio-stakes PRD as long as the phase-blockers are flagged as such.

**Phase-blocker assessment of the assumptions and open questions:**
- **OQ-1 (blocked folders), phase-blocker for UX and a thesis risk.** UJ-1 and OQ-1 itself say that Downloads and Desktop are "where the primary user's new files arrive". If they are blocked, Daniel's main folder may be unusable. That would change the entry flow (FR-1, FR-2), possibly the persona's journey, and would revive OQ-1's drag-and-drop question. Architecture can run the spike, but UX should not finalize the entry and error flows until it is answered.
- **FR-14 content-matching rule (AND, whole word plus prefix, no typos), architecture-shaping.** Prefix matching and snippet extraction determine the index structure and what each index entry stores. The default is reasonable. Architecture should treat it as fixed for the MVP, and the PRD should say so.
- **NFR-4 scale targets (10k entries, 1,000 files in 60 s), architecture-shaping but not blocking.** OQ-2 admits that real folder sizes are unknown. Declare them design targets that architecture builds to, not open values.
- **§3 and FR-11 name-over-content rank, not a blocker.** It is tunable and has a validation path through OQ-3 and SM-1.
- **FR-19 "not reconfirmed", low UX risk.** It affects tree/result layout. Confirm it before UX starts.
- The remaining items (FR-10 thresholds, FR-16 10 MB, FR-20 limits, NFR-6, SM targets) are safe defaults that can be tuned later.

### Findings
- **medium** Phase-blockers aren't marked as such (§9, §10). OQ-1 is listed alongside low-stakes questions such as OQ-6 (name availability), with nothing indicating that it gates UX. *Fix:* add a "Blocks" column or tag to the Open Questions (OQ-1 blocks UX entry and error flows, OQ-2 informs NFR-4), and state in §10 that the FR-14 and NFR-4 defaults are fixed design targets for architecture.
- **low** Location is relative, but the resolution needs an absolute path (UJ-1 Resolution, §3 Location, FR-19). Daniel "notes the file's location and opens it in his usual app", but Location is "relative to the selected folder", and browsers don't expose the selected folder's absolute path. The journey's last step depends on the user remembering where the selected folder is. *Fix:* state this limit in §7.2 or the UJ-1 edge cases (for example, Find shows the selected folder's name as the root of every location), so UX doesn't promise a full path.

## Downstream usability — adequate

The Glossary exists, is well built, and declares "Do not introduce synonyms". FR, NFR, and SM IDs are contiguous and unique, and the features cross-reference UJ-1 steps precisely. The weak point is Glossary discipline. Several load-bearing terms used in FRs are either undefined or synonyms of Glossary terms. Story extraction will copy them as written.

### Findings
- **medium** Glossary drift: synonyms and undefined terms used in FRs (multiple):
  - **"content-only match"** (§3 Rank, FR-11, OQ-3). It is used inside the Rank definition but not defined. The Glossary defines **Content match**, and "content-only" (content match and not a name match) is a different set.
  - **"match kind"** (FR-11). It is undefined, yet the label values include **content**, which is not one of the three Name-match kinds.
  - **"start state"** (FR-2, FR-3). It is undefined, and FR-3 partly defines it inline.
  - **"folder listing" / "a folder's file listing"** (FR-7). This is a synonym for the expanded contents of the folder tree.
  - **"preview panel"** (FR-20, EARS-02) is a synonym for Preview. **"selection" / "selected file entry"** (FR-15, FR-20) is undefined.
  - **"text-based PDFs"** (§1, §4.4, §7.1, NFR-4) and **"scanned PDFs"** (§3, FR-16, UJ-1) are synonyms for "PDF that has (or lacks) a text layer". Pick one pair and define it.
  - **"folder access"** (§2.2, NFR-3, OQ-5) is used alongside "read-only access" (FR-1).
  - **"result"** is used throughout to mean "a file entry in the result list".

  If a match is only a file entry for which a name match or content match is found, also define whether a file entry that is *both* a name match and a content match is labelled with its name-match kind. Nothing currently says so. *Fix:* add Glossary entries for Content-only match, Match kind (exact | partial | typo | content), Start state, and Selected file entry. Replace "folder listing" with "folder tree" and "preview panel" with "preview". Standardize on "PDF with a text layer" and "PDF without a text layer" (or define "Text-layer PDF").
- **low** EARS-01 to EARS-03 point to an external source that isn't linked (FR-8, FR-12, FR-20). The IDs come from an EARS requirements document (`docs/product/earsRequirments`), but §0 doesn't cite it, and EARS-02 and EARS-03 appear without context. *Fix:* cite the source document in §0.
- **low** The Type definition is vague at the edges (§3 Type, Text file). "Plain-text code files" has no extension list, and the Glossary doesn't say whether SVG counts as an image. The addendum excludes SVG from preview, so is it type image (and in the filter) or type other? *Fix:* list the extensions per type in one place, and say "SVG = other in the MVP".

## Shape fit — strong

A single-persona, single-journey consumer utility needs exactly one rich UJ, and that is what it has. The edge cases inside UJ-1 cover the main failure paths without adding journey theater. The length is about right for the agreed 5–8 pages. The addendum keeps API names (`showDirectoryPicker`, `AbortError`, pdf.js, Web Workers, `FileSystemDirectoryHandle`) out of the main body, so FRs leak almost no implementation detail. NFR-8 (above) and the "in-memory store" in the Content index definition are the only real leaks, and the second is a deliberate product constraint (statelessness). No over-formalization.

### Findings
- **low** §0 promises that features are "written around user journeys (§2.3)", but FR-6, FR-7, and FR-17 don't trace to any UJ step. This is fine for this scope. *Fix:* soften §0 to "most features realize UJ-1", or ignore.

## Mechanical notes

- **Assumptions Index roundtrip broken.** §10's first entry cites "the defaults for A1–A3, A5, and A6", but no A-numbered items exist in the PRD, A4 is skipped, and UJ-1 carries no inline `[ASSUMPTION]` tag. Either inline-tag them or point to where A1–A6 are defined (presumably `.memlog.md` or the brief).
- **Rank is defined twice.** The §3 Rank and FR-11 consequence 1 restate the same order, and both are tagged `[ASSUMPTION]`. They are consistent today, but that duplication invites drift. Keep the definition in §3 and have FR-11 reference it.
- **Blocked folder hedge mismatch.** §3 says "possibly Downloads", while UJ-1's edge case says "for example Downloads" as if confirmed. Align them with OQ-1's uncertainty.
- **Privacy scope mismatch.** NFR-1 says "Once Find has loaded … zero network requests", while SM-5 says "zero network requests … during a full session". State that SM-5 measures after load.
- **Session vs reload.** The Glossary Session ends on "tab closed or a different folder chosen". FR-3 tests page reload, which isn't in the definition. Add "or the page is reloaded".
- **FR-4 coverage.** The consequence covers Firefox and Safari only. Mobile Chrome and Edge (excluded in §2.2 and NFR-7) have no defined behavior.
- **IDs.** FR-1 to FR-20, NFR-1 to NFR-8, SM-1 to SM-6, and SM-C1 to SM-C2 are contiguous, with no duplicates. UJ-1 is the only UJ and has a named protagonist. All FR range references in §7.1 resolve.
- **Required sections** are present for the stakes and product type.
- **Brief sync.** OQ-7 says the brief still lists version clusters and the smart date filter in the MVP. Resolve this before downstream workflows source-extract from both documents.
