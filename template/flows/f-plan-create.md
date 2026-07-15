# f-plan-create

---
type: flow
version: x.y
primitives: [READ, WRITE, PRESENT, ASK, SET]
sequences: [s-UPDATE-ISSUE-STATUS]
flows: [f-plan-execute]
updated: YYMMDD-HHMMSS
---

## Purpose

Draft a plan file for any workId. Handles three workTypes:
- `spec` — feature or fix targeting existing packages
- `package` — new package creation
- `issue-bucket` — group of related findings from issues.md

Branching occurs only at step 4 (context loading). All other steps are identical regardless of workType.

## Preconditions

- `work.workId` is set.
- `work.workType` is set: `spec` | `package` | `issue-bucket`.
- Work-pool entry exists for this workId.
- No file matching `plans/[work.workId]-*.md` exists.

## Parameters

```
workType: spec | package | issue-bucket   (from work.workType)
```

## Sequence

```
1. READ data/work-pool.md
   Locate entry for work.workId.
   Confirm work.workType matches entry type.

2. PRESENT exactly:
     "Work-pool entry [work.workId]:
      title:    [title]
      type:     [workType]
      packages: [packages or F-IDs]
      version:  [target-version]"

3. READ data/architecture.md

— STEP 4: context loading — branches on workType —

4. IF workType == spec:
     FOR EACH package in entry.packages:
       READ data/packages/[package]/inner-context.md   (if exists)
       IF work involves public type changes or new exports:
         READ data/packages/[package]/io-schema.md
     IF entry.packages is empty or "none":
       PRESENT "Skipping package reads — packages: none."

   ELSE IF workType == package:
     READ data/packages/[package]/inner-context.md
     READ data/packages/[package]/io-schema.md
     PRESENT exactly:
       "New package context loaded from [package].
        Layer placement must be confirmed in step 5."

   ELSE IF workType == issue-bucket:
     READ data/issues.md
     FOR EACH F-ID from work.findings:
       Locate finding row in issues.md.
       Note: packages, aspect, priority, todo-ref.
     PRESENT exactly:
       "Issue-bucket constituents:
        [list each: F-ID | title | priority | aspect | packages]"
     Collect union of all affected packages.
     FOR EACH package in union:
       READ data/packages/[package]/inner-context.md   (if exists)
       IF any constituent finding involves public type changes:
         READ data/packages/[package]/io-schema.md

5. PRESENT exactly:
     "Proposed scope:
      owner:          [packages] at layer [N]
      reason:         [why these packages own this work]
      dependencies:   [required deps — available / new]
      blockers:       [list or none]
      multi-package:  [yes/no — if yes, list cross-package contracts]"
   (for issue-bucket: also list F-IDs and their aspects)

— STEP 6: plan sections — branches on workType —

6. Draft plan with sections appropriate to workType.
   All plans include these required sections:
     - frontmatter (type=plan, workId, status=planned, created, updated,
                    packages, target-version)
     - Objective
     - Architecture (package/layer table)
     - Constraints
     - Tasks (table: # | Task | Primitive(s) | Status=not-started | Notes)
     - Required Tests (table: # | Description | Type | Status=not-started)
     - Acceptance

   Additional sections by workType:

   IF workType == spec:
     - Cross-Package Contract (required if multi-package; omit if single)

   IF workType == package:
     - Package Role (layer, what it owns, what it does not own)
     - Layer Constraints (what it may and may not import)
     - Initial Exports (first public API surface)
     - Cross-Package Contract (always required — new package always has contracts)

   IF workType == issue-bucket:
     - Constituent Issues (table: F-ID | title | aspect | packages)
     - Cross-Package Contract (required if issues span multiple packages)
     Tasks table gains column: F-ID-ref (links each task to its finding)
     Required Tests table gains column: F-ID-ref

   (do not omit required sections)

7. PRESENT draft plan in full.
   (do not summarise — present complete plan text)

8. ASK "Approve plan, request changes, or cancel?"
   IF approved:  go to step 9
   IF changes:   collect changes, redraft, return to step 7
   IF cancelled:
     SET session.lastAction "f-plan-create cancelled for workId [work.workId]"
     EXIT (no file written)

9. WRITE plans/[work.workId]-[slug]-[session.sessionId].md
    SET work.planFiles[+] "plans/[filename]"
    SET session.phase 1
    SET session.lastAction "plan created for workId [work.workId]"

    UPDATE data/work-pool.md "entries" —
    set status=planned for work.workId.
    (required — work-pool status must advance from draft to planned
     when a plan is approved. Do not omit.)

    IF workType == issue-bucket:
      For each constituent F-ID:
        s-UPDATE-ISSUE-STATUS [F-ID] in-bucket [work.workId]

    PRESENT exactly:
      "Plan saved: plans/[filename]
       work-pool status: draft → planned
       Ready for f-plan-execute."
    (required — WRITE, three SETs, UPDATE work-pool, PRESENT.
     Do not omit any.)
```

## Exit Conditions

- Plan file written to `plans/`.
- `work.planFiles` updated.
- `session.phase` set to 1.
- `session.lastAction` set.
- For issue-bucket: constituent F-IDs updated to `in-bucket` in issues.md.
- Ready for f-plan-execute.

## Notes

- `inner-context.md` is the primary context source and always read when a
  package is involved. `io-schema.md` is conditional.
- For issue-bucket: the F-ID-ref columns in Tasks and Required Tests trace
  each task back to its constituent finding. This matters for p-session-close
  when issues.md statuses are updated to done.
-  The plan tasks should include creating the four package files (inner-context.md,
  io-schema.md) as explicit tasks.
- Step 7 presents the full plan — do not substitute a summary or outline.
  The human approves what will be written to disk.