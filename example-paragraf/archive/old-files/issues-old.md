# paragraf — Issues

---
type: issues
scope: project
updated: 260502-1000
---

## Purpose

Single source of truth for all codebase findings across all reviewers.
Accumulated across review sessions — do not create per-session files.

Findings are added via `p-scan-review`. Issue-buckets reference findings
by ID. Resolved findings are archived to `archive/issues-archive.md`.

---

## Review Sessions

| session-id | date | reviewer | method | scope | summary |
|---|---|---|---|---|---|
| RS-001 | 260421 | claude.ai | full source read | all packages (0-types → 4b-compile), tests, demo | Architecturally sound. Problems cluster around: incomplete features accepted silently (spaceBefore, hyphenation: false, stretch); module-level singletons with no reset; API surface accumulation (deprecated aliases); overflow accounting reimplemented in compile.ts vs layoutDocument. |
| RS-002 | 260501 | vscode-copilot | full source read + todo-list cross-reference | all packages | Confirmed RS-001 themes. Added false-done identification (#80, #22). Found WASM ratio precision issue, BiDi TS fallback gaps, latent RTL rendering bug, and 6 test coverage gaps. |

---

## Findings

| id | title | priority | aspect | status | packages | reviewer | session | bucket-ref |
|---|---|---|---|---|---|---|---|---|
| F008 | FontDescriptor.stretch stored but never matched in selectVariant | P2 | silent-failure | open | types, compile | vscode-copilot | RS-002 | — |
| F009 | parseParaTag loses precision for ICC v4 piecewise TRC (types 1–4) | P2 | correctness | open | color | vscode-copilot | RS-002 | — |
| F010 | Deprecated widowPenalty/orphanPenalty spread across 3 layers | P3 | api-surface | open | types, typography, shaping-wasm | vscode-copilot | RS-002 | — |
| F011 | applyLigatures/applySingleSubstitution required no-ops on FontEngine | P3 | api-surface | open | font-engine | vscode-copilot | RS-002 | — |
| F012 | ParagraphInput.font required even in spans mode | P3 | api-surface | open | typography | vscode-copilot | RS-002 | — |
| F013 | BiDi TS fallback misses Syriac/Thaana/N'Ko and other RTL scripts | P3 | correctness | open | typography | vscode-copilot | RS-002 | — |
| F014 | WASM path registers all fonts at init regardless of usage | P4 | performance | open | typography | vscode-copilot | RS-002 | — |
| F016 | ICC_CHANNELS Lab entry incomplete in PDF output | P4 | correctness | open | render-pdf | vscode-copilot | RS-002 | — |
| F017 | Latent multi-span RTL segment ordering bug in render-core | P2 | latent-bug | open | render-core | vscode-copilot | RS-002 | — |
| F018 | Test gap: deriveLineWidths survival through composeDocument | P1 | test-coverage | open | typography | vscode-copilot | RS-002 | — |
| F019 | Test gap: clearRenderCaches/clearPdfCaches have no tests | P2 | test-coverage | open | render-core, render-pdf | vscode-copilot | RS-002 | — |
| F021 | Test gap: ICC parseParaTag with type 1–4 parametric curves | P2 | test-coverage | open | color | vscode-copilot | RS-002 | — |
| F022 | Test gap: validateTemplate check 7 has no negative test | P2 | test-coverage | open | template | vscode-copilot | RS-002 | — |
| F023 | Test gap: buildSrgbProfileBytes round-trip not verified | P2 | test-coverage | open | color | vscode-copilot | RS-002 | — |
| F024 | Bundle CMYK profile as @paragraf/profiles package | P2 | feature | open | color | vscode-copilot | RS-002 | — |
| F025 | Rename knuth_plass_wasm.js Rust module (covers shaping + linebreaking) | P3 | feature | open | shaping-wasm | vscode-copilot | RS-002 | — |
| F026 | Kashida justification for Arabic text | P3 | feature | open | linebreak, shaping-wasm | vscode-copilot | RS-002 | — |
| F027 | Inline markup → TextSpan[] parsing in compile layer | P2 | feature | open | compile, typography | vscode-copilot | RS-002 | — |
| F028 | leftskip/rightskip margin glue equivalent | P3 | feature | open | linebreak | vscode-copilot | RS-002 | — |
| F029 | Glyph expansion (HZ/pdfTeX ±0.5%) | P3 | feature | open | render-core | vscode-copilot | RS-002 | — |
| F030 | Document model gaps (anchored objects, tables, footnotes) | P3 | feature | open | typography | vscode-copilot | RS-002 | — |
| F031 | Richer style inheritance (nested styles, GREP styles) | P3 | feature | open | style | vscode-copilot | RS-002 | — |
| F032 | Optional conditionals ({{?subtitle}}) in template DSL | P2 | feature | open | template | vscode-copilot | RS-002 | — |
| F033 | @paragraf/studio status investigation | P3 | feature | open | — | vscode-copilot | RS-002 | — |
| F034 | Selection-overlay hit-testing in render-pdf/selectable.ts | P3 | feature | open | render-pdf | vscode-copilot | RS-002 | — |
| F035 | Additional page sizes (JIS B-series, DL envelope, etc.) | P3 | feature | open | render-pdf | vscode-copilot | RS-002 | — |
| F036 | Document concurrency semantics for compileBatch | P3 | feature | open | compile | vscode-copilot | RS-002 | — |
| F037 | Review unexplored code paths (Rust lib.rs 340–1255, qpdf, tests) | P3 | feature | open | shaping-wasm, render-pdf | vscode-copilot | RS-002 | — |
| F038 | Consider splitting @paragraf/typography into -node and -browser | P3 | feature | open | typography | vscode-copilot | RS-002 | — |

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