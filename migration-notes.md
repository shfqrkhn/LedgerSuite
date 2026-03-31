# Migration Notes

## v0.1.41
- Overhauled UX and CSS styling with premium glassmorphism aesthetics, responsive reflows, and adaptive theming.
- Injected specific contextual inline placeholder instructions for all input elements to prevent writer's block (Palette Mode).
- Refactored Recovery panel into native expandable details element and updated Playwright interaction flows.

## v0.1.40
- Applied `touch-action: manipulation` constraint on all inputs, textareas, and selects to prevent mobile zoom-on-tap regressions.
- Purged temporary server log files and advanced version synchronization.

## v0.1.39
- Reworked the UI into a guided, step-labeled flow with inline instructions for first-time users (no tooltip dependency).
- Hardened responsive behavior across phone/tablet/desktop with improved spacing, typography scaling, and tap-target ergonomics.

## v0.1.38
- Strengthened verify gate to require migration-note headings to remain in descending semantic-version order.
- Prevents stale or out-of-order release notes from passing RC validation.

## v0.1.37
- Strengthened verify gate to require the top migration-notes heading to match the current package version.
- Prevents out-of-order or stale release-note updates from passing RC validation.

## v0.1.36
- Added import validation guard to reject unknown entity stores in snapshot payloads.
- Added acceptance regression coverage for unknown-store import rejection and non-committable state.

## v0.1.35
- Added verify gate to enforce `quality.json` schema integrity (required structure, non-empty gate values, and date format).
- Strengthens release governance by catching malformed quality-ledger edits early.

## v0.1.34
- Extended verify task-ledger checks to enforce required task fields (`id`, `title`, `status`) and allowed status values.
- Prevents malformed task entries from passing release validation.

## v0.1.33
- Added verify gate to fail when `tasks.json` contains duplicate task IDs.
- Prevents silent project-ledger corruption during iterative development.

## v0.1.32
- Added acceptance regression coverage for same-file retry after failed import validation.
- Locks import retry resilience for both failure and success flows.

## v0.1.31
- Optimized case-bundle retrieval by parallelizing IndexedDB reads with `Promise.all`.
- Preserved deterministic row ordering for rendered/exported list sections.

## v0.1.30
- Stabilized case-bundle collection ordering (`evidence`, `assumptions`, `options`, `packHooks`) by `id`.
- Improves render/export/print determinism across environments and sessions.

## v0.1.29
- Added acceptance regression coverage for same-file import retry after a successful commit.
- Locks in import chooser reset behavior to prevent retry-flow regressions.

## v0.1.28
- Added import chooser reset after import attempts (success/failure), enabling immediate same-file re-selection.
- Improves import recovery UX when users need to retry corrected snapshots with the same filename.

## v0.1.27
- Extended integrity repair to dedupe singleton-per-case stores (`reviewMatrices`, `decisionRecords`, `outcomeReviews`, `governanceReviews`).
- Validated singleton duplicate prevention via unique caseId index behavior in acceptance flow.

## v0.1.26
- Added import validation guard for duplicate primary keys within each entity store.
- Added acceptance regression coverage to verify duplicate-key imports are rejected and non-committable.

## v0.1.25
- Hardened ZIP import parsing to reject unsafe entry names containing path semantics (`/`, `\\`, `..`).
- Added acceptance regression coverage for unsafe ZIP entry-name rejection.

## v0.1.24
- Added acceptance regression coverage for duplicate singleton import rejection.
- Verifies staged import remains non-committable when duplicate `governanceReviews` rows exist for a case.

## v0.1.23
- Added import validation guard to reject duplicate singleton-per-case records before commit.
- Covered stores: `reviewMatrices`, `decisionRecords`, `outcomeReviews`, and `governanceReviews`.

## v0.1.22
- Updated integrity repair logging to record entries only when invalid records are actually removed.
- Prevents long-term recovery log growth from repetitive zero-removal repair passes.

## v0.1.21
- Strengthened RC self-check to require `touch-action: manipulation` on all buttons, not just the first button.
- Improves mobile interaction safety coverage and prevents partial CSS regressions from passing RC.

## v0.1.20
- Tightened acceptance gate: RC output must contain no `FAIL:` entries.
- Prevents false positives where one gate passes but others silently fail.

## v0.1.19
- Added verify gate requiring `migration-notes.md` to include an entry for the current package version.
- Strengthened release hygiene to prevent undocumented functional/version iterations.

## v0.1.18
- Increased IndexedDB `DB_VERSION` to `3` to introduce `packHooks.caseIdHookType` compound index for efficient upsert lookups.
- Added integrity-repair dedupe pass to collapse legacy duplicate pack hooks by case + hook type.
- Updated pack-hook save path to use compound-index lookup with fallback for backward compatibility.

## v0.1.17
- Added acceptance regression coverage for oversized JSON import rejection.
- Verifies oversized import files remain non-committable in the staged import flow.

## v0.1.16
- Added pre-read import rejection for unsupported file types and oversized files.
- Expanded acceptance checks to verify unsupported import type rejection and disabled commit state.

## v0.1.15
- Extended verify version sync to enforce `tasks.json` and `quality.json` alignment with `package.json`.
- Added release ledger tracking for multi-file version-sync enforcement.

## v0.1.14
- Updated pack hook save behavior to upsert by case + hook type, preventing duplicate growth over time.
- Added acceptance coverage for governance save and pack-hook upsert behavior.

## v0.1.13
- Added one-command RC pipeline script (`npm run rc:full`) that runs verify, full QA, and GitHub Pages build in sequence.
- Synced release docs and quality/task ledgers for the RC pipeline capability.

## v0.1.12
- Tightened import type handling to accept only JSON/ZIP uploads.
- Switched JSON import size guard from character count to UTF-8 byte count.
- Hardened ZIP parsing to require embedded JSON filename.
- Added acceptance regression check that malformed ZIP uploads are rejected and cannot be committed.

## v0.1.11
- Hardened ZIP import parsing with size cap, ZIP flag checks, truncation checks, and CRC32 verification.
- Expanded browser acceptance suite to validate ZIP import commit path in addition to ZIP export.
- Extended verify gate to enforce service worker cache-name version sync with `package.json`.

## v0.1.10
- Added ZIP snapshot export and ZIP import parsing (stored-file method).
- Added Governance Review and Pack Hooks records to local schema, export/import, and printable/markdown briefs.
- Added OPFS artifact persistence for JSON/ZIP/Markdown export outputs (when supported by browser).
- Added guards to prevent saves when no case is selected.

## v0.1.9
- Added mandatory pre-reset backup export prior to local data wipe.
- Added second confirmation step after backup generation before destructive reset.

## v0.1.8
- Added explicit label for snapshot import file input to resolve accessibility sweep failure.

## v0.1.7
- Added automated accessibility sweep script (`qa:a11y`) for keyboard/label/landmark checks.
- Added print brief visual regression script (`qa:print`) with baseline hash tracking.
- Stabilized print brief rendering by removing generated timestamp from printable content.

## v0.1.6
- Enforced integrity metadata as mandatory for schema v2 imports.
- Prevents checksum bypass by rejecting v2 payloads without integrity block.

## v0.1.5
- Added deterministic integrity checksum to snapshot exports (`fnv1a-32`).
- Added checksum verification during import validation when integrity metadata is present.
- Kept backward compatibility with schema v1 and v2 import payloads.

## v0.1.4
- Added import migration compatibility for schema `1` snapshots to current schema `2`.
- Legacy imports are normalized to include `recoveryLogs` before validation/commit.
- Tightened import pipeline to validate only normalized current-schema payloads.

## v0.1.2
- Database version increased to `2`.
- Schema version increased to `2`.
- Added `recoveryLogs` store for integrity and migration events.
- Added integrity repair pass for malformed and orphaned records.
- Added recovery actions for local reset and app cache reset.

## v0.1.1
- Introduced IndexedDB-backed local schema with stores:
  - meta
  - workspaces
  - cases
  - evidenceItems
  - assumptions
  - optionSets
  - reviewMatrices
  - decisionRecords
  - outcomeReviews
- Schema version set to `1` and persisted in `meta`.
- Added deterministic snapshot export/import support.
- Added staged import preview and atomic commit process.

## Forward Rules
- Every schema bump must increment `SCHEMA_VERSION` in `app.js`.
- Migration logic must remain idempotent and preserve existing records.
- Import validation must reject unsupported schema versions before commit.
