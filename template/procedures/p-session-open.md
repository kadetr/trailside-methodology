# p-session-open

---
type: procedure
version: x.y
primitives: [READ, SET, UPDATE]
sequences: [s-INIT-LOG]
updated: YYMMDD-HHMMSS
---

## Purpose

Initialise session state for a workId. Sets sessionId, logFile,
work status, phase. Updates work-pool to in-progress. Pure state
operation — no judgment, no ASK.

## Parameters

- `workId: string` — the workId to open

## Preconditions

- `state/current.yaml` session.* and snapshot.* are clean (reset by
  p-session-continue or fresh schema).
- `workId` is set
- Work-pool entry exists for this workId.

## Sequence

```
1. SET session.sessionId [timestamp YYMMDD-HHMMSS]
   SET session.logFile "logs/session-log-[sessionId].yaml"
   SET work.workId [workId]
   SET session.loopType "inner"

2. s-INIT-LOG(sessionId: session.sessionId, workId: work.workId)

3. READ data/work-pool.md
   Locate entry for work.workId.
   SET work.workType [entry.type]
   SET work.packages [entry.packages]
   SET work.findings [entry.findings]

4. SET session.lastAction "session-open complete"

5. PRESENT exactly:
     "Session open.
      workId:  [work.workId]
      type:    [work.workType]
      phase:   [session.phase]
      log:     [session.logFile]"
```

## Exit Conditions

- `session.sessionId` set.
- `session.logFile` set, log file created via s-INIT-LOG.
- `work.workId`, `work.workType`, `work.packages`, `work.findings` set.
- `session.lastAction` set.
- `session.phase` is set by p-session-continue (resume) or defaults to 1 (fresh).