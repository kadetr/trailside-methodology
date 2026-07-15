# f-start

---
type: flow
version: 5
calls: [READ, ASK, PRESENT, SET]
invokes: [p-session-continue, p-session-open, f-plan-create, f-plan-execute]
updated: 260502-1800
---

## Purpose

Session entry point. Owns all human judgment for session initialisation.
Routes to correct session procedure, then hands off to execution.

## Human invocation

```
Run methodology/flows/f-start.md
```

## Sequence

```
1. READ state/current.yaml
   READ data/work-pool.md

2. IF snapshot.nextAction is non-empty (previous session exists):
     PRESENT exactly:
       "Previous session found.
        workId:     [snapshot.workId]
        nextAction: [snapshot.nextAction]
        incomplete: [snapshot.incompleteSteps]"
     ASK "Continue workId [snapshot.workId], or pick different work?"

     IF continue:
       READ methodology/procedures/p-session-continue.md
       Execute p-session-continue(retainedWorkId: snapshot.workId)
       (p-session-continue sets work.workId and session.phase from snapshot)
       go to step 4 — workId already set, skip step 3

     IF different:
       READ methodology/procedures/p-session-continue.md
       Execute p-session-continue(retainedWorkId: null)
       (p-session-continue clears snapshot, work.workId = null)
       go to step 3

   ELSE (fresh start — no snapshot):
     SET session.phase 1
     go to step 3

3. PRESENT work-pool entries with status in [draft, planned, in-progress]
   ASK "Which workId to work on?"
   SET work.workId [chosen]

4. ASK "Verbose mode? (yes / no)"
   SET session.verbose [true | false]

5. READ methodology/procedures/p-session-open.md
   Execute p-session-open(workId: work.workId)
   (p-session-open: sets sessionId, logFile, work fields, updates work-pool)

6. IF plans/[work.workId]-*.md does not exist:
     PRESENT exactly:
       "No plan for workId [work.workId]. workType: [work.workType]"
     ASK "Create plan now?"
     IF yes:
       READ methodology/flows/f-plan-create.md
       Execute f-plan-create from step 1
     IF no:
       ASK "Stop session, or proceed without plan?"
       IF stop: EXIT

7. READ plans/[work.workId]-*.md
   PRESENT exactly:
     "Plan: [plan title]
      Done:    [count] tasks
      Pending: [not-started task count]
      Next:    [next pending task description]
      verbose: [session.verbose]
      Ready for f-plan-execute."

8. READ methodology/flows/f-plan-execute.md
   Execute f-plan-execute from step 1.
   (f-plan-execute exits via s-CLOSE-SESSION at every exit point)
```

## Lifecycle diagram

```
f-start
  ├─ [snapshot + continue]  → p-session-continue(retain) ─┐
  ├─ [snapshot + different] → p-session-continue(null) → ASK workId ─┐
  └─ [fresh start]          → SET phase=1 → ASK workId ──────────────┤
                                                           p-session-open
  ├─ [no plan]  → f-plan-create ─┐
  └─ [plan exists] ──────────────┤
                          f-plan-execute
                    ├─ [tasks remain] → loop
                    ├─ [revise]       → f-plan-create → return
                    └─ [done/paused]  → s-CLOSE-SESSION → EXIT
```

## Exit Conditions

- p-session-close has run via s-CLOSE-SESSION.
- Snapshot written to state/current.yaml.
- Session log written and closed.