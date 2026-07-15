# Primitives — Verb Vocabulary

---
type: primitive
version: x.y
updated: YYMMDD-HHMMSS
---

## Index

`READ` `WRITE` `UPDATE` `DELETE` `SET` `RESET` `ASK` `PRESENT` `VERIFY`

---

## Layer Restrictions

| Primitive | Flows | Procedures | Sequences |
|---|---|---|---|
| READ | ✓ | ✓ | ✓ |
| WRITE | ✓ | ✓ | ✓ |
| UPDATE | ✓ | ✓ | ✓ |
| DELETE | ✗ | ✗ | ✓ |
| SET | ✓ | ✓ | ✗ |
| RESET | ✗ | ✓ | ✗ |
| VERIFY | ✓ | ✓ | ✓ |
| PRESENT | ✓ | ✓ | ✗ |
| ASK | ✓ | ✗ | ✗ |

ASK is the only source of human judgment. It lives in flows only.
Sequences are silent — no ASK, no PRESENT, no SET, no RESET.

---

## READ

Load a file into context.

- inputs: `path: string`
- outputs: file content in context
- side effects: none

---

## WRITE

Create a new file or overwrite an existing one in full.

- inputs: `path: string, content: string`
- outputs: none
- side effects: file created or fully overwritten on disk

---

## UPDATE

Modify one named section of an existing file.
Section identified by markdown heading. `"frontmatter"` targets the YAML
block above the first heading.

- inputs: `path: string, section: string, content: string`
- outputs: none
- side effects: named section replaced on disk

---

## DELETE

Remove a file from disk permanently. Irreversible.
Only call after VERIFY confirms the archive destination exists.
Never call on `state/current.yaml`, `data/`, or any methodology file.

- inputs: `path: string`
- outputs: none
- side effects: file removed from disk

---

## SET

Write one field to `state/current.yaml` via dotted path.
The ONLY primitive that writes individual fields to `current.yaml`.
Append to list fields using `[+]` suffix.

- inputs: `field: string (dotted path), value: any`
- outputs: none
- side effects: field written to `state/current.yaml`

---

## RESET

Clear one section of `state/current.yaml` to empty schema values.
The ONLY primitive that clears sections of `current.yaml`.

- inputs: `scope: session | work | snapshot | all`
- outputs: none
- side effects: named section(s) overwritten with empty schema values

Scopes:
- `session`  — clears session.*
- `work`     — clears work.*, document.*
- `snapshot` — clears snapshot.*
- `all`      — clears all sections

---

## ASK

Block execution until human responds.

- inputs: `question: string`
- outputs: `answer: string`
- side effects: none until answer received

---

## PRESENT

Output content to human without blocking.

- inputs: `content: string`
- outputs: none
- side effects: none

A terminal completion point is the final output of a procedure or a
meaningful state change confirmation. Procedures must not use PRESENT
for intermediate progress, step narration, or enforcement.

---

## VERIFY

Assert a condition. Does not block — the caller decides what to do with
the result.

- inputs: `condition: string`
- outputs: `{ result: pass | fail, detail: string }`
- side effects: none

---

## Constraints

- `PRESENT` in procedures is restricted to terminal completion points only.
  Sequences must never call PRESENT.
- `DELETE` is restricted to sequences (via s-MOVE only). Never call directly from flows or procedures.
- `SET` and `RESET` are the only primitives that write to `state/current.yaml`.
- Session log is written by s-WRITE-LOG sequence at task completion.
- Every primitive call must specify all inputs explicitly. No defaults.
- Primitives do not call other primitives. Composition is the responsibility of flows and procedures.