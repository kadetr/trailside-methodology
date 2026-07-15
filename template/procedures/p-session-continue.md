# p-session-continue

---
type: procedure
version: x.y
primitives: [READ, SET, RESET]
updated: YYMMDD-HHMMSS
---

## Purpose

Consume and clear snapshot state from a previous session. Prepares
clean state for p-session-open. Owns all RESET calls in the methodology.

## Parameters

- `retainedWorkId: string | null`
  — workId to continue, or null if human chose different work

## Preconditions

- `state/current.yaml` snapshot fields are populated.

## Sequence

```
1. READ state/current.yaml

2. IF retainedWorkId is not null:
     SET work.workId [retainedWorkId]
     (work.workType is set by p-session-open from work-pool — not set here)

3. Capture phase before RESET wipes snapshot:
   SET session.phase [snapshot.phase]
   (required — snapshot.phase is derived by p-session-close from actual
    plan state, so it is reliable. Capture it now before RESET fires.)

4. RESET snapshot
   RESET session
   (required — both RESETs. Clears stale state from previous session.
    work.workId survives because it is in work.*, not session.* or snapshot.*.
    session.phase survives because it was SET in step 3 before RESET session —
    wait: RESET session clears session.phase too.)

   SET session.phase [value captured in step 3]
   (required — restore phase after RESET session clears it)

5. PRESENT exactly:
     "Previous session cleared.
      Retained workId: [work.workId or 'none — fresh start']
      Resuming phase:  [session.phase]"
```

## Exit Conditions

- `snapshot.*` cleared.
- `session.*` cleared then session.phase restored.
- `work.workId` set if retainedWorkId was provided, null otherwise.
- `session.phase` set from snapshot.phase (reliable — written by p-session-close).
- Ready for p-session-open.

## Notes

- RESET snapshot and RESET session are the only RESET calls in the
  entire methodology. No other procedure or flow calls RESET.
- work.workId is SET before RESET fires. RESET clears session.* and
  snapshot.* — work.* is not in scope for RESET work here.
- session.phase is captured before RESET and restored after. This avoids
  p-session-open needing to re-read the plan file to derive phase.
- snapshot.phase is written by p-session-close from actual plan state,
  not from session.phase — so it is reliable to reuse here.
- work.workType is not set here — p-session-open reads it from work-pool.