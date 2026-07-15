# f-group-issues

---
type: flow
version: 2
calls: [READ, UPDATE, PRESENT, ASK, SET]
updated: 260502-1100
---

## Purpose

Analyse open findings in issues.md and group related ones into an
issue-bucket work-pool entry. Judgment-heavy — requires analysis of
finding relationships, package overlap, and risk profile. Human
confirms grouping before the bucket is created.

## Preconditions

- `data/issues.md` exists with at least two findings with `status: open`.
- `data/work-pool.md` exists.
- `work.workId` is not required — this flow runs outside a workId context.

## Sequence

```
1. READ data/issues.md
   Extract all rows where status = open.
   PRESENT exactly:
     "Open findings:
      [list each: F-ID | title | priority | aspect | packages]"
   IF fewer than 2 open findings:
     PRESENT "Not enough open findings to form a bucket. Exiting."
     EXIT

2. READ data/work-pool.md
   Note the highest existing workId number in the entries table.
   Retain: highestWorkId = [highest workId found]
   Filter open findings to exclude those already with status = in-bucket.
   PRESENT exactly:
     "Highest existing workId: [highestWorkId]
      Available for bucketing:
      [list: F-ID | title | priority | aspect | packages]"

3. READ data/architecture.md

4. Analyse available findings in this order:

   a. Priority filter:
      Group P0/P1 findings first — these are highest urgency.
      P2 findings next. P3/P4 last.
      Do not mix priority bands unless package overlap is exact and
      scope is clearly bounded.

   b. Package grouping (primary structural criterion):
      Within the same priority band, group findings by package overlap.
      Findings touching the same package(s) should be in the same bucket.
      A bucket touching  2-4 packages is ideal. 4+ packages increases risk.

   c. Aspect cohesion (secondary criterion):
      Within the same package group, prefer findings with the same aspect.
      Do not sacrifice package cohesion for aspect cohesion.

   d. Risk profile (gate — must pass before proposing):
      Non-feature: safe = silent-failure, api-surface, test-coverage, documentation
                   separate = correctness, latent-bug, technical-debt with algorithm changes
      Feature: safe = additive single-package, investigation, naming
               separate = new package, multi-package with contracts, algorithm changes
      Never mix feature and non-feature findings in the same bucket.

   e. Context reuse check:
      Confirm the proposed bucket benefits from shared package context
      in the conversation summary. If findings touch unrelated packages,
      split into separate buckets even if aspect matches.

5. PRESENT proposed grouping exactly:
     "Proposed bucket:
      Title:     [one-line bucket title]
      F-IDs:     [list: F-ID | title | aspect]
      Packages:  [union of affected packages]
      Rationale: [why these belong together]
      Risk:      [low / medium]
      Excluded:  [findings not included and why]"
   (Do not propose groupings that mix high-risk correctness bugs with
    routine silent-failure or api-surface items. Keep risk homogeneous.
    Do not mix feature findings with non-feature findings.)

6. New bucket workId = highestWorkId + 1.
   (do not start from 1 — always continue from the highest workId
    already in the entries table, regardless of entry status)
   PRESENT exactly:
     "Highest existing workId: [highestWorkId]
      New bucket workId:       [highestWorkId + 1]
      Title:    [title]
      F-IDs:    [list]
      Packages: [list]"
   ASK "Confirm workId [highestWorkId + 1] creation?"
   IF no: return to step 4

7. UPDATE data/work-pool.md "entries" — add new row:
     workId=[highestWorkId+1] | type=issue-bucket | title=[title] |
     status=draft | packages=[union] | target-version=— |
     created=[ts] | updated=[ts]

8. SET work.workId [highestWorkId+1]
   SET work.workType "issue-bucket"

   PRESENT exactly:
     "Issue-bucket workId [highestWorkId+1] created.
      F-IDs assigned: [list]
      Note: F-ID statuses remain 'open' until f-plan-create runs
      and updates them to 'in-bucket'.
      Next: run f-work-run to open a session for this workId."
```

## Exit Conditions

- New issue-bucket entry in work-pool with status=draft.
- `work.workId` and `work.workType` set.
- F-IDs in issues.md NOT yet updated — that happens in f-plan-create step 10.

## Notes

- Do NOT group findings with different risk profiles.
  Do NOT group feature findings with non-feature findings.
- After a bucket is created, the constituent findings remain open in
  issues.md until f-plan-create runs. This is intentional — the bucket
  can be cancelled before planning begins without orphaning in-bucket statuses.