# paragraf — Issues

---
type: issues
version: 3
updated: 260502-1000
---

## Purpose

Single source of truth for all codebase findings across all reviewers.
Accumulated across review sessions — do not create per-session files.

Findings are added via `p-scan-review`. Issue-buckets reference findings
by ID. Resolved findings are archived to `archive/issues-archive.md`.

The `bucket` and `parallel` columns are human-maintained only.
No flow or procedure writes to them.

---

## Review Sessions

| session-id | date | reviewer | method | scope | summary |
|---|---|---|---|---|---|
| RS-001 | 260421 | claude.ai | full source read | all packages (0-types → 4b-compile), tests, demo | Architecturally sound. Problems cluster around: incomplete features accepted silently (spaceBefore, hyphenation: false, stretch); module-level singletons with no reset; API surface accumulation (deprecated aliases); overflow accounting reimplemented in compile.ts vs layoutDocument. |
| RS-002 | 260501 | vscode-copilot | full source read + todo-list cross-reference | all packages | Confirmed RS-001 themes. Added false-done identification (#80, #22). Found WASM ratio precision issue, BiDi TS fallback gaps, latent RTL rendering bug, and 6 test coverage gaps. |

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
open → in-bucket  (assigned by f-group-issues + confirmed in f-plan-create)
     → wont-fix   (human decision)

in-bucket → done  (fix verified after bucket completes)
          → open  (bucket cancelled or finding split out)

false-done → open    (reclassified after source inspection)
           → done    (actually fixed in a subsequent session)

done / wont-fix → archive/issues-archive.md  (via p-archive-items Layer 5)
```