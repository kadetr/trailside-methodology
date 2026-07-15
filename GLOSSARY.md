---

## Glossary

Every term below is traced to the file that defines it. Regenerated from
`template/` — if a definition and the source file disagree, the source file
is authoritative.

### 1. Layers

| Term | Definition |
|---|---|
| **primitive** | `primitives.md` — one of 9 abstract verbs (`READ`, `WRITE`, `UPDATE`, `DELETE`, `SET`, `RESET`, `ASK`, `PRESENT`, `VERIFY`). Fixed signature, no branching, calls nothing else. The only layer with no judgment and no composition. |
| **sequence** | `sequences.md` — a named, silent composition of 3+ primitive calls, used to eliminate drift on a repeating pattern (e.g. `sq-MOVE`, `sq-WRITE-LOG`). Sequences take explicit parameters, never branch on judgment, and never touch state (`SET`/`RESET`) or the human (`ASK`/`PRESENT`). |
| **procedure** | `procedures/` — a linear, non-branching sequence used for routines like session bookkeeping (open, close, archive). May call primitives and sequences, never other procedures or flows, and never `ASK`. May branch only on data conditions. |
| **flow** | `flows/` — the only layer allowed to exercise human judgment, via `ASK`. Invoked directly by a human or by another flow; orchestrates primitives, sequences, procedures, and other flows. Owns branching on interpretation or ambiguity. |

### 2. State & session concepts

| Term | Definition |
|---|---|
| **state** | `state/current.yaml` — the single live, mutable record of the current session and work item. Written only through `SET` (one field at a time, dotted path) and `RESET` (clears a whole section). Never written directly by `WRITE`/`UPDATE`. |
| **session** | One continuous run of the methodology, from `p-session-open` to `p-session-close`. Identified by `session.sessionId` (a `YYMMDD-HHMMSS` timestamp), tracked through `session.phase`, `session.condition`, `session.lastAction`, and backed by a per-session log file at `session.logFile`. |
| **phase** | Where a session sits in its current task. Per `f-plan-execute.md`: `1` = read/scaffolding, `2` = test-writing, `3` = implementation. Stored at `session.phase`, and re-derived (not copied) into `snapshot.phase` at session close. |
| **snapshot** | The cold-start recovery record, `snapshot.*` in `state/current.yaml`. Written at session close (`p-session-close`), consumed and cleared at the next session's continuation (`p-session-continue`). Captures `completedSteps`, `incompleteSteps`, `pendingChecks`, `knownIssues`, and `nextAction`, so a fresh session can resume without replaying prior context. |
| **work** | The `work.*` section of state describing the item currently being worked on (`workId`, `workType`, `status`, `packages`, `planFiles`, `blockers`, `findings`). Persists across sessions until the work completes; cleared by `RESET work`. |
| **task** | One row in a plan's Tasks table — the smallest unit `f-plan-execute` operates on. Has a description, associated primitive(s), a status (`not-started` → `in-progress` → `done`), and for issue-bucket plans, an `F-ID-ref`. Executed strictly one at a time — no batching. |

### 3. Work Items & Planning

| Term | Definition |
|---|---|
| **work-pool** | `data/work-pool.md` — the master table of all work items. Status flow: `draft` → `planned` → `in-progress` → `human-review` → `done` \| `deferred` \| `cancelled` (`deferred` reachable only from `in-progress`; `done` is terminal). The entry point for choosing what to work on next (`f-work-run` reads this to select a `workId`). |
| **type** (work) | `data/work-pool.md`'s `type` field — a closed set: `spec` (feature/fix targeting existing packages), `package` (new package creation), or `issue-bucket` (a group of related findings). Determines which branch `f-plan-create` takes at context-loading and plan-section steps. |
| **plan** | `plans/[workId]-[slug]-[sessionId].md` — drafted by `f-plan-create`, approved by the human via `ASK` before being written. Required sections: frontmatter, Objective, Architecture, Constraints, Tasks, Required Tests, Acceptance — plus a `Cross-Package Contract` section (required for multi-package `spec` work, always for `package` work, and for multi-package `issue-bucket` work). |

### 4. Findings / Issues

| Term | Definition |
|---|---|
| **finding** | A row in `data/issues.md`'s Findings table — the single source of truth for codebase problems across all reviewers, accumulated (never reset) over time. Columns: `id`, `title`, `priority`, `aspect`, `status`, `packages`, `reviewer`, `session`, `bucket-ref`, `bucket`, `parallel`. The `bucket`/`parallel` columns are human-maintained only — no flow or sequence writes to them. |
| **priority** | A finding's severity, `P0`–`P4` (set by `sq-FORMAT-ISSUE`): `P0` correctness bug or data loss, `P1` reliability/test-isolation/divergence risk, `P2` silent failure/latent bug/important test gap, `P3` API-surface/consistency, `P4` performance/technical-debt/minor gaps. |
| **aspect** | A finding's category, from the closed Aspect Vocabulary in `data/issues.md`: `correctness`, `reliability`, `silent-failure`, `api-surface`, `latent-bug`, `performance`, `technical-debt`, `documentation`, `test-coverage`, `feature`. Distinct from `priority` — aspect is *what kind* of problem, priority is *how bad*. |
| **status** (finding) | `open` → `in-bucket` (assigned to a work-pool `issue-bucket` entry) → `done` (fix verified) — or `open` → `wont-fix`. `false-done` is a reclassification state for findings marked done in a todo list but unchanged in behavior. |

### 5. Architecture & Context

| Term | Definition |
|---|---|
| **architecture** | `data/architecture.md` — the package/layer dependency map (outer-loop) plus the io-schema navigation table (inner-loop, on-demand). Defines which layer a package sits at and what it may import from. |
| **inner-context** | `data/packages/[name]/inner-context.md` — per-package purpose, role, and ownership boundaries ("responsible for" / "does not own"). Always read when a package is involved in planning; the primary context source in `f-plan-create`. |
| **io-schema** | `data/packages/[name]/io-schema.md` — full field-by-field type and export definitions for one package's public surface. Conditional read: only loaded when the work involves public type changes or new exports. Indexed by `data/io-schemas.md`. |

### 6. Archive

| Term | Definition |
|---|---|
| **archive** | `archive/` — destination for completed history, moved out of active files by `p-archive-items` across four layers: work-pool entries (`archive/work-pool-archive.md`), plan files (`archive/plans/`, via `sq-MOVE`), session logs (`archive/logs/`, via `sq-MOVE`), and issues (`archive/issues-archive.md`, with `resolved-by` and `archived` fields appended). Triggered by status = `done` or `cancelled`/`wont-fix`, never run mid-session. |
