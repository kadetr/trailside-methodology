# Issues

---
type: issues
scope: project
updated: [YYMMDD-HHMMSS]
---

## Purpose

Single source of truth for all codebase findings across all reviewers.
Accumulated across review sessions — do not create per-session files.

Findings are added during review work. Issue-buckets reference findings
by ID. Resolved findings are archived to `archive/issues-archive.md`.

The `bucket` and `parallel` columns are human-maintained only.
No flow or sequence writes to them.

---

## Review Sessions

| session-id | date | reviewer | method | scope | summary |
|---|---|---|---|---|---|

---

## Findings

| id | title | priority | aspect | status | packages | reviewer | session | bucket-ref | bucket | parallel |
|---|---|---|---|---|---|---|---|---|---|---|

---

## Aspect Vocabulary

| aspect | meaning |
|---|---|
| correctness | wrong output — behaviour differs from spec or documented intent |
| reliability | test isolation, state divergence, singleton risk |
| silent-failure | feature accepted by API, cascaded, but has no runtime effect |
| api-surface | deprecated, inconsistent, or confusing public API |
| latent-bug | currently guarded but dangerous if guard is removed |
| performance | unnecessary startup cost, inefficiency |
| technical-debt | duplication, workaround, parallel reimplementation |
| documentation | JSDoc wrong, misleading, or missing |
| test-coverage | missing test for existing or fixed behaviour |
| feature | new capability identified during review; not yet planned |

<uncertain class="shape-or-stub">This vocabulary was authored for a TypeScript library project (JSDoc, singleton risk). The categories generalize reasonably to most codebases, but the wording is domain-flavored. Kept as shape — a closed vocabulary is required for sq-FORMAT-ISSUE determinism — reword entries for your domain if needed.</uncertain>

---

## Status Vocabulary

| status | meaning |
|---|---|
| open | finding confirmed, not yet assigned to a bucket |
| in-bucket | assigned to an issue-bucket workId (see bucket-ref) |
| done | fix verified — ready to archive |
| wont-fix | accepted as by-design or deferred indefinitely |
| false-done | marked done in todo list but behaviour unchanged |

---

## Lifecycle

```
open → in-bucket  (assigned during issue-bucket planning)
     → wont-fix   (human decision)

in-bucket → done  (fix verified after bucket completes)
          → open  (bucket cancelled or finding split out)

false-done → open    (reclassified after source inspection)
           → done    (actually fixed in a subsequent session)

done / wont-fix → archive/issues-archive.md  (via p-archive-items Layer 4)
```

<uncertain class="residue">Source said "p-archive-items Layer 5" — but p-archive-items.md only defines four layers (work-pool, plans, logs, issues), with issues as Layer 4. This looks like a genuine off-by-one in the source, not a rename artifact. Corrected to Layer 4 here; flagging rather than silently fixing since it's evidence of the drift pattern your Article III documents, not a derivation choice.</uncertain>
