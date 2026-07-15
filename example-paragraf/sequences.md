# Sequences — Concrete Operation Bindings

---
type: sequence
version: 4
updated: 260502-1600
---

## Purpose

Sequences are named compositions of primitives bound to specific patterns.
Where primitives are abstract verbs (READ, WRITE, DELETE), sequences are
concrete operations with named roles (source, destination, id).

A sequence reduces drift by collapsing a multi-step primitive sequence into
one named call. The sequence defines the sequence; the caller provides
the parameters.

Sequences are silent — no ASK, no PRESENT, no SET, no RESET.

---

## When to Add a Sequence

Add a sequence when:
- A primitive sequence of 3+ steps repeats across multiple procedures
- The sequence has a clear name in plain English (move, format, update)
- Drift is observed on any step in the sequence

Do not add sequences speculatively. Derive from observed usage.

---

## Sequence Definitions

---

### s-MOVE

Move a file from source path to destination path.
Verifies the destination before deleting the source.

**Parameters:**
- `source: string` — path of the file to move
- `dest: string` — destination path (must include filename)

**Sequence:**
```
1. READ [source]
2. WRITE [dest] [content from step 1]
3. VERIFY "[dest] exists and content matches source"
   IF fail: EXIT sequence — source not deleted
4. DELETE [source]
```

**Guarantees:**
- Source is never deleted unless destination is verified.
- If WRITE or VERIFY fails, source remains untouched.

**Examples:**
- `s-MOVE plans/006-cache-key-260430-1400.md archive/plans/006-cache-key-260430-1400.md`
- `s-MOVE logs/session-log-260430-1400.yaml archive/logs/session-log-260430-1400.yaml`

**Do not use s-MOVE on:**
- `state/current.yaml`
- Any file in `data/` (data files are updated in place, not moved)
- Methodology files (index.md, primitives.md, sequences.md, procedures, flows)

---

### s-FORMAT-ISSUE

Format a raw finding into a structured issues.md Findings table row.
Silent — returns candidate row to caller. Caller handles confirmation.

**Parameters:**
- `raw: string` — the raw finding text from the review file

**Sequence:**
```
1. From raw text, extract:
     title:    short phrase (max 10 words) describing the finding
     priority: P0 / P1 / P2 / P3 / P4
               P0: correctness bug or data loss
               P1: reliability, test isolation, divergence risk
               P2: silent failure, latent bug, important test gap
               P3: API surface, consistency
               P4: performance, technical debt, minor gaps
     aspect:   from Aspect Vocabulary in data/issues.md
     packages: affected packages (short-names, comma-separated)

2. Return candidate row:
     { title, priority, aspect, packages, bucket: —, parallel: — }
     (id, status, reviewer, session, bucket-ref set by caller;
      bucket and parallel are human-maintained — leave as — on creation)
```

**Output:** Candidate row — caller presents and confirms before writing.

---

### s-UPDATE-ISSUE-STATUS

Update the status and bucket-ref fields for one finding in issues.md.

**Parameters:**
- `id: string` — the F-ID to update (e.g. F006)
- `status: string` — new status value
- `bucket-ref: string` — workId or — (use — if status is not in-bucket)

**Sequence:**
```
1. UPDATE data/issues.md "Findings" —
   find the row where id = [id].
   Set status = [status].
   Set bucket-ref = [bucket-ref].
```

**Valid status transitions:**
- `open` → `in-bucket`  (bucket-ref = workId)
- `open` → `wont-fix`   (bucket-ref = —)
- `in-bucket` → `done`  (bucket-ref unchanged)
- `in-bucket` → `open`  (bucket-ref = —, if bucket cancelled)
- `false-done` → `open` (bucket-ref = —)
- `false-done` → `done` (bucket-ref = workId or —)

---

### s-WRITE-LOG

Read the existing log file, append new entries, write the full updated
file. Keeps the log as a complete accumulated record.

**Parameters:** `entries: list of { ts, primitive, target, result }`

**Sequence:**
```
1. READ [session.logFile]
2. Append [entries] to existing list
3. WRITE [session.logFile] complete updated content
```

Entry format:
```yaml
- ts: YYMMDD-HHMMSS
  primitive: READ | WRITE | UPDATE | DELETE | SET | RESET | VERIFY | PRESENT | ASK
  target: file path, dotted field, condition, or question
  result: pass | fail | info
```

---

### s-INIT-LOG

Create a new session log file with frontmatter and empty entries list.

**Parameters:** `sessionId: string, workId: string | null`

**Sequence:**
```
1. WRITE logs/session-log-[sessionId].yaml exactly:
     "---
      type: session-log
      session: [sessionId]
      workId: [workId]
      status: open
      ---
      []"
```

