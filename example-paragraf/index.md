# paragraf — Methodology Index

---
type: methodology
version: 3
updated: 260502-1500
---

## Purpose

Entry point for every session. Defines layer contracts, load order, vocabulary index, ask-human triggers, and constraints.

---

## Load Order

```
1. READ state/current.yaml
2. READ primitives.md
3. READ sequences.md
4. SELECT a workId from data/work-pool.md — choose any entry with
   status: draft, planned, or in-progress.
   READ methodology/flows/f-work-run.md
   Execute f-work-run(workId: [selected workId]) from step 1.
```

---

## Layer Contracts

| Layer | ASK | PRESENT | Judgment | Calls |
|---|---|---|---|---|
| Primitive | defined | defined | none | nothing |
| Sequence | no | never | none | primitives only |
| Procedure | no | yes - results | none - conditions | primitives + sequences |
| Flow | yes | yes | owner | all layers |

**Data condition** — branching on a file value or state value (IF status == done).
**Judgment** — branching on interpretation, ambiguity, or human preference. Flows only.

---

## Layer Map

| Layer | Path | Lifecycle | Purpose |
|---|---|---|
| Index | `index.md` | read once per session | entry point, load order, triggers |
| Primitives | `primitives.md` | read once per session | abstract verb primitives |
| Sequences | `sequences.md` | read once per session | concrete operation bindings |
| Data | `data/` | on-demand | read-mostly project knowledge |
| State | `state/current.yaml` | per-step | live mutable session state |
| Plans | `plans/` | per-work | per-workId execution plan |
| Procedures | `procedures/` | on-demand | linear sequences (no branching) |
| Flows | `flows/` | on-demand | branching / iterating workflows |
| Logs | `logs/` | per-task | structured session audit trail |

---

## Primitive Index

Full definitions in `primitives.md`.

| Primitive | Purpose | Caller restriction |
|---|---|---|
| `READ` | load file into context | any |
| `WRITE` | create or overwrite file | sequences, procedures, flows |
| `UPDATE` | modify one named section | sequences, procedures, flows |
| `DELETE` | remove file permanently | sequences only |
| `SET` | write field to current.yaml | procedures, flows |
| `RESET` | clear section of current.yaml | procedures only |
| `ASK` | block for human input | flows only |
| `PRESENT` | output to human | flows always; procedures at terminals only |
| `VERIFY` | assert condition | sequences, procedures, flows |

---

## Sequence Index

Full definitions in `sequences.md`.

| Sequence | Parameters |
|---|---|
| `s-MOVE` | `source, dest` |
| `s-WRITE-LOG` | `entries: list` |
| `s-INIT-LOG` | `sessionId, workId` |
| `s-UPDATE-ISSUE-STATUS` | `id, status, bucket-ref` |
| `s-FORMAT-ISSUE` | `raw: string` |

---

## Procedure / Flow Index

| File | Type |
|---|---|---|
| `flows/f-plan-execute.md` | flow |
| `flows/f-plan-create.md` | flow |
| `flows/f-work-run.md` | flow |
| `procedures/p-session-open.md` | procedure |
| `procedures/p-session-continue.md` | procedure |
| `procedures/p-session-close.md` | procedure |
| `procedures/p-archive-items.md` | procedure |

---

## Ask-Human Triggers

Stop and ASK when any condition below is true. Flows only.

| Condition | Question |
|---|---|
| Work spans two or more layers | "This spans layers [N] and [M]. Review architecture?" |
| New dependency not in architecture.md | "Needs [dep] not in architecture.md. Check for conflicts?" |
| About to remove or rename a public type | "Removes/renames [Type]. Check other packages?" |
| Implementation contradicts a package constraint | "Conflicts with constraint: [constraint]. Proceed or revise?" |
| Edge case not in the plan | "Found edge case: [description]. Add to plan?" |
| Uncertainty about package ownership | "Belongs in [A] or [B]?" |
| Scope change mid-implementation | "Scope changed: [what]. Stop and revise?" |

---

## Frequency Vocabulary

| Value | Meaning |
|---|---|
| `per-step` | updated at every primitive call during session |
| `per-session` | read at open, written at close |
| `per-work` | durable across sessions until work completes |
| `on-demand` | loaded only when specifically needed |
---

## Constraints

- `ASK` is restricted to flows. Never in procedures or sequences.
- `PRESENT` in procedures is restricted to terminal completion points.
  Never in sequences.
- `DELETE` is restricted to sequences (called only via s-MOVE).
- `RESET` is called only from procedures.
- Session log is written only by s-WRITE-LOG.
- Every primitive call specifies all inputs explicitly. No defaults.
- Tasks in f-plan-execute execute one at a time. No batching.
- A judgment call requires ASK. Never decide independently.