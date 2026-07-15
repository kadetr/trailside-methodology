# paragraf — Issues Archive

---
type: issues-archive
scope: project
updated: 260502-1000
---

## Purpose

Archived findings from issues.md. Append-only.
Active findings live in `data/issues.md`.

When a finding is archived:
1. Copy row from issues.md to this file's Entries table
2. Add `resolved-by` and `archived` fields
3. UPDATE issues.md — remove the row

---

## Entries

| id | title | priority | aspect | status | packages | reviewer | session | todo-ref | bucket-ref | resolved-by | archived |
|---|---|---|---|---|---|---|---|---|---|---|---|
| F006 | spaceBefore/spaceAfter silently ignored at runtime | P2 | silent-failure | done | style, compile | vscode-copilot | RS-002 | — | 010 | workId 010 | 260501-1900 |
| F007 | hyphenation: false silently ignored at runtime | P2 | silent-failure | done | style, compile | vscode-copilot | RS-002 | — | 010 | workId 010 | 260501-1900 |
| F001 | composeDocument ignores styleDefaults.lineWidth | P0 | correctness | done | typography | claude.ai | RS-001 | — | 101 | workId 101 | 260501-1455 |
| F003 | Module-level singletons; no reset mechanism for tests | P1 | reliability | done | typography, compile | vscode-copilot | RS-002 | — | 101 | workId 101 | 260501-1455 |
| F002 | singleLinePenalty fires on wrong condition; JSDoc inaccurate | P0 | documentation | done | linebreak | vscode-copilot | RS-002 | — | 102 | workId 102 | 260502-1100 |
| F020 | Test gap: singleLinePenalty correctness not verified | P2 | test-coverage | done | linebreak | vscode-copilot | RS-002 | — | 102 | workId 102 | 260502-1100 |
| F010 | Deprecated widowPenalty/orphanPenalty spread across 3 layers | P3 | api-surface | done | types, typography, shaping-wasm | vscode-copilot | RS-002 | — | 105 | workId 105 | 260503-1553 |
| F011 | applyLigatures/applySingleSubstitution required no-ops on FontEngine | P3 | api-surface | done | font-engine | vscode-copilot | RS-002 | — | 105 | workId 105 | 260503-1553 |
| F012 | ParagraphInput.font required even in spans mode | P3 | api-surface | done | typography | vscode-copilot | RS-002 | — | 105 | workId 105 | 260503-1553 |
| F004 | Overflow count reimplemented in compile.ts vs layoutDocument | P1 | technical-debt | done | compile | vscode-copilot | RS-002 | — | 103 | workId 103 | 260502-1200 |
| F015 | compile.ts step 9 never calls deriveLineWidths | P4 | technical-debt | done | compile | vscode-copilot | RS-002 | — | 103 | workId 103 | 260502-1200 |
| F005 | WASM ratio clamp is JS-side workaround for Rust precision bug | P1 | latent-bug | done | shaping-wasm | vscode-copilot | RS-002 | — | 104 | workId 104 | 260503-0000 |
| F009 | parseParaTag loses precision for ICC v4 piecewise TRC (types 1–4) | P2 | correctness | done | color | vscode-copilot | RS-002 | — | D | workId 108 | 260503-1630 |
| F016 | ICC_CHANNELS Lab entry incomplete in PDF output | P4 | correctness | done | render-pdf | vscode-copilot | RS-002 | — | D | workId 108 | 260503-1630 |
| F019 | Test gap: clearRenderCaches/clearPdfCaches have no tests | P2 | test-coverage | done | render-core, render-pdf | vscode-copilot | RS-002 | — | C | workId 107 | 260503-1700 |
| F021 | Test gap: ICC parseParaTag with type 1–4 parametric curves | P2 | test-coverage | done | color | vscode-copilot | RS-002 | — | C | workId 107 | 260503-1700 |
| F022 | Test gap: validateTemplate check 7 has no negative test | P2 | test-coverage | done | template | vscode-copilot | RS-002 | — | C | workId 107 | 260503-1700 |
| F023 | Test gap: buildSrgbProfileBytes round-trip not verified | P2 | test-coverage | done | color | vscode-copilot | RS-002 | — | C | workId 107 | 260503-1700 |
| F014 | WASM path registers all fonts at init regardless of usage | P4 | performance | done | typography | vscode-copilot | RS-002 | 106 | B | workId 106 | 260503-1730 |
| F018 | Test gap: deriveLineWidths survival through composeDocument | P1 | test-coverage | done | typography | vscode-copilot | RS-002 | 106 | B | workId 106 | 260503-1730 |
| F008 | FontDescriptor.stretch stored but never matched in selectVariant | P2 | silent-failure | done | types, compile | vscode-copilot | RS-002 | 109 | E | workId 109 | 260503-1800 |
| F017 | Latent multi-span RTL segment ordering bug in render-core | P2 | latent-bug | done | render-core | vscode-copilot | RS-002 | 109 | E | workId 109 | 260503-1800 |
| F031 | Richer style inheritance (nested styles, GREP styles) | P3 | feature | done | style | vscode-copilot | RS-002 | 114 | J | workId 114 | 260503-1850 |
| F032 | Optional conditionals ({{?subtitle}}) in template DSL | P2 | feature | done | template | vscode-copilot | RS-002 | 114 | J | workId 114 | 260503-1850 |
| F013 | BiDi TS fallback misses Syriac/Thaana/N'Ko and other RTL scripts | P3 | correctness | done | typography | vscode-copilot | RS-002 | 110 | F | workId 110 | 260503-1910 |
| F025 | Rename knuth_plass_wasm.js Rust module (covers shaping + linebreaking) | P3 | feature | done | shaping-wasm | vscode-copilot | RS-002 | 112 | H | orange | workId 112 | 260503-2000 |
| F026 | Kashida justification for Arabic text | P3 | feature | done | linebreak, shaping-wasm | vscode-copilot | RS-002 | 112 | H | orange | workId 112 | 260503-2000 |
| F028 | leftskip/rightskip margin glue equivalent | P3 | feature | done | linebreak | vscode-copilot | RS-002 | 112 | H | orange | workId 112 | 260503-2000 |
| F029 | Glyph expansion (HZ/pdfTeX ±0.5%) | P3 | feature | done | render-core | vscode-copilot | RS-002 | 112 | H | orange | workId 112 | 260503-2000 |

---

## resolved-by vocabulary

- `workId [N]` — the bucket that fixed this finding
- `by-design` — accepted as intentional behaviour
- `superseded-by F[N]` — a later finding covers this one
- `todo-only` — fix tracked only in todo-list, not in a bucket| F027 | Inline markup → TextSpan[] parsing in compile layer | P2 | feature | done | compile, typography | vscode-copilot | RS-002 | 111 | G | white | 111 | 260503-2010 |
| F036 | Document concurrency semantics for compileBatch | P3 | feature | done | compile | vscode-copilot | RS-002 | 111 | G | white | 111 | 260503-2010 |
| F034 | Selection-overlay hit-testing in render-pdf/selectable.ts | P3 | feature | done | render-pdf | vscode-copilot | RS-002 | 113 | I | white | 113 | 260503-2020 |
| F035 | Additional page sizes (JIS B-series, DL envelope, etc.) | P3 | feature | done | render-pdf | vscode-copilot | RS-002 | 113 | I | white | 113 | 260503-2020 |
| F024 | Bundle CMYK profile as @paragraf/profiles package | P2 | feature | done | color | vscode-copilot | RS-002 | 115 | K | white | 115 | 260503-2100 |
| F030 | Document model gaps (anchored objects, tables, footnotes) | P3 | feature | done | typography | vscode-copilot | RS-002 | 115 | K | white | 115 | 260503-2100 |
| F033 | @paragraf/studio status investigation | P3 | feature | done | — | vscode-copilot | RS-002 | 115 | K | white | 115 | 260503-2100 |
| F037 | Review unexplored code paths (Rust lib.rs 340–1255, qpdf, tests) | P3 | feature | done | shaping-wasm, render-pdf | vscode-copilot | RS-002 | 115 | K | white | 115 | 260503-2100 |
| F038 | Consider splitting @paragraf/typography into -node and -browser | P3 | feature | done | typography | vscode-copilot | RS-002 | 115 | K | white | 115 | 260503-2100 |
