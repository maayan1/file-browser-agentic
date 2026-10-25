# Review Polish Log: PRD Find

Date: 2026-10-25. Review: `bmad-review` with the structure lens, then the prose lens. Reader: humans. Style guide: Microsoft Writing Style Guide.

The PRD is for the people who design, architect, and build Find, and for the downstream BMAD workflows. It helps them build the MVP from testable requirements. The PRD's structure (Vision, Users, Glossary, Features, NFRs, Scope, Metrics, Open Questions, Assumptions) already fits the requirements-spec model. The structure lens found no cuts or merges worth making in `prd.md`. It marked the Product Principles and the Glossary as PRESERVE.

A check after editing confirmed that every ID (FR-, NFR-, SM-, UJ-, OQ-, EARS-), number, threshold, `[ASSUMPTION]` / `[NOTE FOR PM]` tag, and the YAML frontmatter match the originals.

## Edits applied (10)

### prd.md (7)

1. §1 Vision: changed "PDFs with a text layer" to "text-layer PDFs", the Glossary term.
2. §3 Glossary, Selected folder: removed "There is one per session." It repeated "The one folder the user chooses in a session."
3. FR-1: changed "an explicit user action, a click on **Choose folder**" so that a colon comes before "a click", which makes it clear that the click is the action.
4. FR-13: changed "progress as files indexed out of the total" to "progress as the number of files indexed out of the total".
5. FR-20: changed "For type other" to "For the type **other**". The type name is now formatted like the other type names.
6. NFR-7: turned the sentence fragment into a full sentence ("Find supports the current and previous major versions…") and removed a stray comma.
7. §7.1 and §7.2: added a blank line after each heading, as the rest of the document does.

### addendum.md (3)

8. Intro: turned the fragment into a full sentence ("This addendum holds technical detail…").
9. Changed "e.g. pdf.js" to "for example, pdf.js", as the Microsoft style guide asks.
10. Structure (MOVE): moved **Name-date detection** from "Rejected alternatives" to "Likely implementation notes". It is guidance for building the detector, not a rejected alternative. The wording did not change.

## Findings not applied (they would change meaning)

1. **The addendum contradicts itself and FR-2.** The Folder access note says "FR-2 needs a way to tell these apart" (a cancelled picker vs. a blocked folder). But FR-2 and the addendum's own rejected alternative say they cannot be told apart and are handled the same way. Suggested fix: change the note to "FR-2 treats them the same." The PM should confirm.
2. **The "Blocks" marking and OQ IDs are inconsistent.** §0 says questions that block a later phase are marked **Blocks** in §9. But OQ-6 ("Confirm before UX") and the FR-19 `[ASSUMPTION]` ("confirm before UX") look like they block UX and are not marked **Blocks**. Also, the §9 items are a plain numbered list, while the rest of the document cites them as OQ-1 and so on. Labeling them explicitly would add IDs, so that change was not made.
3. **Two labels for content that isn't searched.** FR-16 uses **content not searchable** for most cases and **content not searched (format not supported yet)** for DOCX and XLSX. If these are meant to be two separate UI labels, add both to the Glossary. If they are meant to be one label, choose one.
4. Minor: the document uses the British spelling "labelled", while the Microsoft style guide uses "labeled". The spelling is used the same way throughout, so it was left as is. §7.2 "Conversational or AI search. Vision." is too terse to tell whether it means "deferred to the long-term vision". This needs the PM's intent, so it was not reworded.
