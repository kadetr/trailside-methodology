# f-work-run

---
type: flow
version: x.y
primitives: [READ, PRESENT, SET, VERIFY]
procedures: [p-session-continue, p-session-open, p-session-close, p-archive-items]
flows: [f-plan-create, f-plan-execute]
updated: YYMMDD-HHMMSS
---

## Purpose

Non-interactive session entry point. Takes a workId parameter and runs
the full session lifecycle without routing ASKs. Handles all exit signals
from f-plan-execute and runs post-session cleanup.

Planning ASKs (scope confirmation, plan approval) still fire if a plan
does not exist.

## Human invocation

```
Run methodology/flows/f-work-run.md workId=666
```

## Parameters

- `workId: string` — the workId to execute

## Sequence

```
1. READ state/current.yaml
   READ data/work-pool.md

   VERIFY "work-pool entries table contains workId [workId]"
   IF fail:
     PRESENT "workId [workId] not found in work-pool. Exiting."
     EXIT

   SET session.verbose false

2. IF snapshot.workId == workId AND snapshot.nextAction is non-empty:
     PRESENT exactly:
       "Resuming previous session.
        workId:     [snapshot.workId]
        nextAction: [snapshot.nextAction]
        incomplete: [snapshot.incompleteSteps]"
     READ methodology/procedures/p-session-continue.md
     Execute p-session-continue(retainedWorkId: workId)

   ELSE IF snapshot.workId != workId AND snapshot.nextAction is non-empty:
     (stale snapshot from a different workId — clear it before proceeding)
     READ methodology/procedures/p-session-continue.md
     Execute p-session-continue(retainedWorkId: null)
     SET session.phase 1
     SET work.workId [workId]

   ELSE:
     SET session.phase 1
     SET work.workId [workId]

3. READ methodology/procedures/p-session-open.md
   Execute p-session-open(workId: work.workId)

4. IF plans/[work.workId]-*.md does not exist:
     PRESENT "No plan for workId [work.workId]. workType: [work.workType]"
     READ methodology/flows/f-plan-create.md
     Execute f-plan-create from step 1

5. READ plans/[work.workId]-*.md
   PRESENT exactly:
     "Plan: [plan title]
      Done:    [count] tasks
      Pending: [not-started task count]
      Next:    [next pending task description]
      Ready for f-plan-execute."

6. READ methodology/flows/f-plan-execute.md
   Execute f-plan-execute from step 1.
   (f-plan-execute returns control here via EXIT signal)

   SET session.condition [session.lastAction]
   (capture exit signal now — p-session-close overwrites session.lastAction in
    its step 6. session.condition persists until RESET session fires at the
    start of the next session.)

7. READ methodology/procedures/p-session-close.md
   Execute p-session-close from step 1.

   IF session.condition == "verify-fail-revise":
     READ methodology/flows/f-plan-create.md
     Execute f-plan-create (revise mode) from step 1.
     return to step 6

8. READ methodology/procedures/p-archive-items.md
   Execute p-archive-items from step 1.
   (required unconditionally — p-archive-items is a no-op when there is
    nothing to archive. Never skip this step regardless of session.condition.)

9. IF session.condition == "plan-complete":
     PRESENT exactly:
       "Session complete. workId: [workId]"
   ELSE IF session.condition == "task-skipped":
     PRESENT exactly:
       "Session paused — task skipped by human.
        Resume: Run f-work-run.md workId=[snapshot.workId]
        Next action: [snapshot.nextAction]"
   ELSE IF session.condition == "verify-fail-stop":
     PRESENT exactly:
       "Session stopped — verify failure.
        Resume: Run f-work-run.md workId=[snapshot.workId]
        Next action: [snapshot.nextAction]"
```

## Lifecycle diagram

```
f-work-run(workId)
  ├─ [snapshot matches] → p-session-continue → p-session-open
  └─ [fresh]            → p-session-open

  ├─ [no plan]    → f-plan-create
  └─ [plan exists]
        f-plan-execute ─────────────────────────────────────────────┐
          ├─ plan-complete      → p-session-close → p-archive-items │
          ├─ task-skipped       → p-session-close ─────────── EXIT  │
          ├─ verify-fail-stop   → p-session-close ─────────── EXIT  │
          └─ verify-fail-revise → p-session-close                   │
                                  f-plan-create (revise) ───────────┘
```

## Exit Conditions

- p-session-close has run.
- p-archive-items has run for plan-complete only.
- Snapshot written to state/current.yaml.
- Session log written and closed.

## Notes

- f-work-run removes routing ASKs only. Planning ASKs in f-plan-create
  still fire when a new plan is needed.
- If snapshot exists for a different workId it is ignored — snapshot
  check only applies when snapshot.workId matches the parameter.