# p-archive-items

---
type: procedure
version: 5
primitives: [READ, WRITE, UPDATE, PRESENT, VERIFY]
sequences: [s-MOVE]
updated: 260502-2100
---

## Purpose

Move completed and cancelled items out of active files into archive.
Covers four layers: work-pool entries, plan files, session logs, issues.

## Preconditions

- `archive/` folder exists with subfolders `plans/`, `logs/`.
- `archive/work-pool-archive.md` exists (may be empty).
- `archive/issues-archive.md` exists (may be empty).
- Session log files have frontmatter (created by p-session-open v2+).

## Sequence

```
1. READ data/work-pool.md
   READ data/issues.md

   Scan all four layers and collect candidates:

   Layer 1 — work-pool entries with status = done OR cancelled:
     candidates.workPool = [list: workId | title | status]

   Layer 2 — plans/ files with frontmatter status = done OR cancelled:
     READ each file in plans/ one by one, check frontmatter status.
     candidates.plans = [list: filename | status]

   Layer 3 — logs/ files with frontmatter status = closed:
     READ each file in logs/ one by one, check frontmatter status.
     candidates.logs = [list: filename]

   Layer 4 — issues.md rows with status = done OR wont-fix:
     candidates.issues = [list: id | title | status | bucket-ref]

2. PRESENT exactly:
     "Archive candidates:

      Work-pool entries ([count]):
      [list: workId | title | status]

      Plan files ([count]):
      [list: filename | status]

      Session logs ([count]):
      [list: filename]

      Issues ([count]):
      [list: id | title | status]

      Nothing to archive: [layer names with zero candidates, or 'none']"

— LAYER 1: work-pool entries —

3. FOR EACH entry in candidates.workPool:
     READ archive/work-pool-archive.md
     IF file does not exist: WRITE it with header:
       "# Work Pool Archive
        ---
        type: archive
        updated: [ts]
        ---
        ## entries
        | workId | type | title | status | packages | created | updated | findings |
        |---|---|---|---|---|---|---|---|"
     WRITE archive/work-pool-archive.md — append row:
       Table structure: header row, separator row, data rows.
       New row appends after all existing data rows.
       Never insert between header and separator.
     UPDATE data/work-pool.md "entries" — remove the archived row.

   VERIFY "data/work-pool.md entries table has no done or cancelled rows"
   IF fail: PRESENT which rows remain and stop.

— LAYER 2: plan files —

4. FOR EACH file in candidates.plans:
     s-MOVE plans/[filename] archive/plans/[filename]

— LAYER 3: session logs —

5. FOR EACH file in candidates.logs:
     s-MOVE logs/[filename] archive/logs/[filename]

— LAYER 4: issues —

6. FOR EACH issue in candidates.issues:
     READ archive/issues-archive.md
     WRITE archive/issues-archive.md — append row:
       All columns from issues.md row +
       resolved-by: [workId from bucket-ref, or "by-design" if wont-fix]
       archived:    [ts]
     UPDATE data/issues.md "Findings" — remove the archived row.

   VERIFY "data/issues.md Findings table has no done or wont-fix rows"
   IF fail: PRESENT which rows remain and stop.

7. PRESENT exactly:
     "Archive complete.
      Work-pool entries archived: [count]
      Plan files archived:        [count]
      Session logs archived:      [count]
      Issues archived:            [count]
      Active work-pool remaining: [list titles or 'none']"
```

## Exit Conditions

- All done/cancelled work-pool entries in `archive/work-pool-archive.md`.
- All completed plan files in `archive/plans/` (source deleted).
- All closed session logs in `archive/logs/` (source deleted).
- All done/wont-fix issues in `archive/issues-archive.md`.

## Notes

- s-MOVE guarantees destination exists before deleting source.
- If any archive file does not exist, it is created with the appropriate
  header before the first row is written.