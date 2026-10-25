<!-- bmad:context -->
<!-- Verified 2026-10-25 against 3cf6e64. Managed by bmad-project-context; edits inside this block are replaced on refresh. Keep anything you want preserved outside the markers. -->

## Find

Find is a browser-only, read-only local file search app for desktop Chrome and Edge. The project is greenfield: there is no application code yet, and the stack will be chosen in the BMAD architecture step. Requirements live in the PRD at `_bmad-output/planning-artifacts/prds/prd-file-brwsr-2026-10-25/prd.md`. Where the PRD and the brief (`_bmad-output/planning-artifacts/briefs/brief-file-brwsr-2026-10-25/`) disagree, the PRD wins.

## Policy

- Never write, rename, move, or delete user files. Request folder access in read mode only (PRD NFR-2).
- Never make network requests after page load: no analytics, CDNs, or remote fonts at runtime. Bundle every dependency instead (NFR-1).
- Never put file content, metadata, folder handles, queries, or filters in localStorage, IndexedDB, Cache Storage, or cookies. Keep session data in memory only (NFR-3).
- Keep matching, ranking, filtering, name-date detection, and indexing rules unit-testable without rendering the UI (NFR-8).
- Never edit `_bmad/config.toml`, because the BMAD installer regenerates it. Put overrides in `_bmad/custom/config.toml`.
- Never hand-edit `.claude/skills/` or anything in `_bmad/` outside `_bmad/custom/`: they are installer-managed. Reinstall BMAD to restore or update them.

## Where things are

- Domain terms: use the PRD Glossary (§3) terms exactly in code, tests, and stories, with no synonyms.
- Technical notes for architecture: `_bmad-output/planning-artifacts/prds/prd-file-brwsr-2026-10-25/addendum.md`.
- EARS acceptance tests: `docs/product/earsRequirments`. EARS-01, EARS-02, and EARS-03 map to FR-8, FR-20, and FR-12.
- Decision logs: `.memlog.md` in each planning folder. They are append-only: add entries with `python _bmad/scripts/memlog.py append --workspace <folder> --type <type> --text "<entry>"`, and never edit them by hand.

## Running and verifying

- TODO: build, test, and lint commands. They don't exist until `bmad-architecture` chooses the stack. Add them here then, and don't invent them before that.

## Conventions that differ from defaults

- Name the product "Find" in UI, code, and docs. "File Brwsr" is only the BMAD config label.

<!-- /bmad:context -->
