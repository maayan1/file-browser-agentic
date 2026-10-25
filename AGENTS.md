<!-- bmad:context -->
<!-- Verified 2026-10-26 against f0f6934 plus the uncommitted planning updates of 2026-10-25/26. Managed by bmad-project-context; edits inside this block are replaced on refresh. Keep anything you want preserved outside the markers. -->

## Find

Find is a browser-only, read-only, filename-only local file finder for desktop Chrome and Edge. The project is greenfield: there is no application code yet. Requirements live in the PRD at `_bmad-output/planning-artifacts/prds/prd-file-brwsr-2026-10-25/prd.md`, where §11 holds deferred requirements that are not in the MVP. The approved architecture is `_bmad-output/planning-artifacts/architecture/architecture-file-brwsr-2026-10-25/ARCHITECTURE-SPINE.md`. Where the brief disagrees with them, the PRD and the spine win.

## Policy

- Never start feature work before the spike items in the spine (S3, S4, S6, S7, S8) have passed (AD-19).
- Follow the spine's AD rules. Never contradict one locally. A change needs a new or amended AD, recorded in the architecture decision log.
- Never write, rename, move, or delete user files, and never read file contents. Request folder access in read mode only (NFR-2, AD-15).
- Never make network requests after app-ready: no analytics, CDNs, remote fonts, or inlined `data:` assets. Bundle every dependency instead (NFR-1, AD-14).
- Never put file content, metadata, folder handles, queries, or filters in localStorage, sessionStorage, IndexedDB, Cache Storage, OPFS, cookies, or the URL. Keep session data in memory only (NFR-3, AD-16).
- Keep matching, filtering, result ordering, and message selection in `src/core/`, unit-testable without rendering the UI (NFR-8, AD-3).
- Never edit `_bmad/config.toml`, because the BMAD installer regenerates it. Put overrides in `_bmad/custom/config.toml`.
- Never hand-edit `.claude/skills/` or anything in `_bmad/` outside `_bmad/custom/`: they are installer-managed. Reinstall BMAD to restore or update them.

## Where things are

- Domain terms: use the PRD Glossary (§3) and the spine's AD-9 types exactly in code, tests, and stories, with no synonyms.
- The UI talks to the core only through the `SessionController` interface in `src/core/index.ts` (AD-9).
- EARS acceptance tests: `docs/product/earsRequirments`. EARS-01 maps to FR-8; EARS-03 (rev. 3) and EARS-04 (rev. 1) map to FR-12. EARS-02 is deferred with preview (PRD §11, FR-20).
- Decision logs: `.memlog.md` in each planning folder. They are append-only: add entries with `python _bmad/scripts/memlog.py append --workspace <folder> --type <type> --text "<entry>"`, and never edit them by hand.

## Running and verifying

- TODO: no commands exist yet. The stack is decided in the spine's *Stack* table: npm, Vite 8 (`react-ts` starter), TypeScript 6.0.3, Vitest 5, Playwright 1.64, and ESLint 10. When `package.json` exists, record here the exact build, test, lint, and e2e commands. Don't guess them.
- Run CSP and network checks against the production build (`vite build` + `vite preview` under the `/<repo>/` base), never against the dev server (AD-14, AD-20).

## Conventions that differ from defaults

- Pin TypeScript 6.0.3, not npm's `latest` 7.x: typescript-eslint and the boundary lint rules don't support TypeScript 7 (spine *Stack*).
- Lint with ESLint, typescript-eslint, and eslint-plugin-boundaries, replacing the `react-ts` starter's oxlint (AD-2, AD-15).
- Name the product "Find" in UI, code, and docs. "File Brwsr" is only the BMAD config label.

<!-- /bmad:context -->
