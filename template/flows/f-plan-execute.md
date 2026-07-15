# f-plan-execute

---
type: flow
version: x.y
primitives: [READ, WRITE, UPDATE, PRESENT, ASK, SET, VERIFY]
procedures: [p-session-close, p-archive-items]
flows: [f-plan-create, f-plan-execute]
updated: YYMMDD-HHMMSS
---

## Purpose

Execute tasks from the active plan one at a time. Returns control to
caller at every exit point. Caller is responsible for p-session-close
and any post-session steps.

## Preconditions

- `work.workId` is set.
- `session.logFile` is set.
- Active plan loaded in context.
- At least one task has `status: not-started` or `status: in-progress`.

## Exit signals

Caller reads `session.lastAction` after f-plan-execute returns:
- `plan-complete` — all tasks done
- `task-skipped` — human declined a task in verbose mode
- `verify-fail-stop` — VERIFY failed, human chose stop
- `verify-fail-revise` — VERIFY failed, human chose revise

## Session Log Pattern

After each task completes (step 7), READ existing log, append new entries,
WRITE complete file to `[session.logFile]`.

Log entry format:
```yaml
- ts: YYMMDD-HHMMSS
  primitive: READ | WRITE | UPDATE | SET | ASK | PRESENT | VERIFY
  target: file path, dotted field, condition, or question
  result: pass | fail | info
```

## Sequence

```
1. Identify next pending task:
     - `in-progress` tasks first (resume order)
     - then `not-started` tasks in plan order
   IF none:
     PRESENT exactly: "Plan complete. All tasks done."
     SET session.lastAction "plan-complete"
     EXIT — return control to caller

2. PRESENT exactly:
     "Task [#] of [total]: [description]
      Primitive(s): [primitive list from plan]
      Tests:   [associated Required Tests if any]"

3. IF session.verbose == true:
     ASK "Proceed with task [#]?"
     IF no:
       SET session.lastAction "task-skipped"
       EXIT — return control to caller

4. UPDATE plans/[work.workId]-*.md "Tasks" — set task [#] to "in-progress"
   SET session.phase based on task type:
     test-writing:    SET session.phase 2
     implementation:  SET session.phase 3
     READ/scaffolding:SET session.phase 1

   IF work.status != "in-progress":
     SET work.status "in-progress"
     UPDATE data/work-pool.md "entries" — set status=in-progress for work.workId

5. EXECUTE task using primitives only:
     WRITE / UPDATE / READ / ASK as needed.
   No primitive outside primitives.md vocabulary.
   No batching — one task per execution step.
   Track every primitive call in working memory.

6. Run Required Tests for this task:
     FOR EACH associated Required Test:
       VERIFY [condition]
       Track result in working memory.
   IF no Required Tests:
     PRESENT "No Required Tests for task [#] — proceeding to step 6a."

6a. TDD red-phase check — only when session.phase == 2:
   IF session.phase == 2:
     VERIFY "npm run test --workspace=[package] — new tests fail"
     IF all new tests fail:
       PRESENT "✓ Red phase confirmed. Proceed to implementation tasks."
     IF any new test passes:
       return to step 5

7. IF all VERIFY pass:
     UPDATE plans/[work.workId]-*.md "Tasks" — set task [#] to "done"
     UPDATE plans/[work.workId]-*.md "Required Tests" — mark verified tests done
     SET session.lastAction "task [#] done"
     SET snapshot.completedSteps[+] "task [#]: [description]"

     READ [session.logFile]
     Append task entries (primitive calls from step 5, VERIFYs from step 6,
     red-phase from step 6a).
     WRITE [session.logFile] complete updated content.

     IF session.verbose == true:
       PRESENT "✓ Task [#] complete. [N] remaining. Next: [description]"

   IF any VERIFY fail:
     PRESENT "✗ Task [#] failed. [condition] → [detail]"
     ASK "Retry, revise plan, or stop?"
     IF retry:  return to step 5
     IF revise:
       SET session.lastAction "verify-fail-revise"
       EXIT — return control to caller
     IF stop:
       SET session.lastAction "verify-fail-stop"
       EXIT — return control to caller

8. Check ask-human triggers (index.md):
   IF any condition true:
     ASK [question from trigger table]
     apply response

9. IF more pending tasks:
     continue to step 1
   ELSE:
     PRESENT "Plan complete. All tasks done."
     SET session.lastAction "plan-complete"
     EXIT — return control to caller
```

## Exit Conditions

- `plan-complete` — caller runs p-session-close then p-archive-items.
- `task-skipped` — caller runs p-session-close then p-archive-items.
- `verify-fail-stop` — caller runs p-session-close then p-archive-items.
- `verify-fail-revise` — caller runs f-plan-create then re-enters f-plan-execute.

## Notes

- Do not call p-session-close from this flow. Return control to caller.
- Do not advance work-pool to done from this flow. That is p-session-close.
- work-pool in-progress update fires in step 4 on first task execution.
- Log WRITE accumulates all previous entries plus current task entries.
- Do not batch tasks across step numbers.