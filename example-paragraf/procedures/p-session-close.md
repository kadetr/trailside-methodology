# p-session-close

---
type: procedure
version: 3
primitives: [READ, WRITE, UPDATE, SET, RESET, PRESENT, VERIFY]
sequences: [s-UPDATE-ISSUE-STATUS]
updated: 260502-1200
---

## Purpose

Persist session state, write snapshot for cold-start recovery, update
work-pool and plan status, transition issue statuses if issue-bucket,
write final log entry, present summary.

## Preconditions

- `session.sessionId` is set.
- `state/current.yaml` is writable.
- `session.logFile` is set.

## Sequence

```
1. PRESENT exactly:
     "Session summary
      workId:      [work.workId]
      workType:    [work.workType]
      completed:   [snapshot.completedSteps count] tasks this session
      remaining:   [pending task count from plan]
      blockers:    [work.blockers or none]"

2. READ plans/[work.workId]-*.md
   Confirm task statuses are current before writing snapshot.
   Determine actual phase from plan state:
     - any tasks with status in-progress or not-started: use session.phase
     - all tasks done: phase 3 (implementation complete)

3. Determine nextAction:
     - pending tasks exist:  "Resume task [#]: [description]"
     - plan complete:        "Plan complete. Work-pool updated to done."
     - session aborted:      "Investigate [reason] before resuming."

4. SET snapshot.sessionDate           [session.sessionId]
   SET snapshot.workId                [work.workId]
   SET snapshot.phase                 [phase from step 2 — not session.phase directly]
   SET snapshot.incompleteSteps       [in-progress task #s or empty]
   SET snapshot.pendingChecks         [document.pendingChecks or empty]
   SET snapshot.knownIssues           [LLM-flagged issues or empty]
   SET snapshot.nextAction            [from step 3]
   (required — all seven SET calls, do not omit any)
   (YAML constraint — each field must appear exactly once. Write all seven
    in a single pass, top to bottom. Do not write any field twice.)

5. IF all plan tasks are done:
     READ data/work-pool.md
     Check actual current status of work.workId entry.
     PRESENT exactly:
       "Work-pool current status for workId [work.workId]: [actual status]"

     IF actual status is done:
       PRESENT "Work-pool already marked done. Skipping transition."
     ELSE:
       (override any status — draft, planned, or in-progress — to done.
        The session completed all tasks. Status transition is confirmed.)
       UPDATE data/work-pool.md "entries" — set status=done for work.workId
       UPDATE plans/[work.workId]-*.md "frontmatter" — set status=done
       SET work.status "done"
       PRESENT "Work-pool and plan status updated to done."
       (required — both UPDATEs and SET. Do not omit any.)

     IF work.workType == "issue-bucket":
       READ plans/[work.workId]-*.md
       Extract all F-IDs from the Constituent Issues table.
       PRESENT exactly:
         "Updating issue statuses to done: [list F-IDs]"
       For each F-ID:
         s-UPDATE-ISSUE-STATUS [F-ID] done [work.workId]
       (required — all F-IDs must be updated. Do not omit any.)

     RESET work
     RESET snapshot
     (required — clears work.*, document.*)

6. SET session.lastAction "session-close complete"

7. READ existing log content from [session.logFile]
   Update frontmatter and append final entry:
     - Change frontmatter field `status: open` → `status: closed`
     - Append to entries list:
       "- ts: [ts]
          primitive: session-close
          target: session complete
          result: pass"
   WRITE the complete updated file (frontmatter + full entries list).
   (required — this is the final write to the log for this session)

8. IF snapshot.workId is null:
     (done — snapshot was RESET in step 5)
     PRESENT exactly:
       "Session closed.
        Log: [session.logFile]
        Work complete."
   ELSE:
     PRESENT exactly:
       "Session closed.
        Log: [session.logFile]
        Next session:
          1. Run methodology/flows/f-work-run.md workId=[snapshot.workId]
          2. Resume at: [snapshot.nextAction]"
```

## Exit Conditions

- `state/current.yaml` snapshot fields populated (seven fields, each exactly once).
- Plan frontmatter status = done if plan complete.
- work-pool entry status = done if plan complete.
- If issue-bucket: all constituent F-IDs updated to done in issues.md.
- `work.*`, `document.*` reset.
- Session log finalized: final entry written, frontmatter `status: closed`.
- Cold-start instruction visible to human.

## Notes

- Step 5 reads the actual work-pool status before transitioning. It does
  NOT use VERIFY to block on "in-progress" — status may be draft or planned
  due to known drift in p-session-open. The transition to done is always
  applied if all tasks are done, regardless of current status.
- `snapshot.phase` is derived from the plan state in step 2, not copied
  from `session.phase`. This prevents stale phase values.
- The seven snapshot SET calls in step 4 must each appear exactly once.
  YAML does not allow duplicate keys. Write all seven in a single sequential pass.
- RESET work clears `work.*` and `document.*` together.
- Does NOT reset `session.*` — that happens at the start of the next p-session-open.
- If session was aborted, still run p-session-close. Write the abort reason
  as `snapshot.nextAction`. Do not skip snapshot writes.