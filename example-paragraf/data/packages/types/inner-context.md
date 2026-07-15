# types — Inner Context

---
type: inner-context
package: @paragraf/types
updated: 260416-1830
---

## Purpose

Package-level consistency and accuracy checker. Consulted at start and end of every inner-loop task for this package.

## Role

Owns all shared TypeScript interfaces, type aliases, and constants used across the paragraf monorepo. Zero runtime dependencies. Every other paragraf package depends on this package.

Responsible for:
- Font types (`Font`, `FontSpec`, `FontDescriptor`, `FontRegistry`, `FontMetrics`, `FontWeight`, `FontStyle`, `FontStretch`, `FontVariant`, `FontId`)
- Span and segment types (`TextSpan`, `SpanSegment`)
- Line-breaking I/O types (`Node`, `Box`, `Glue`, `Penalty`, `Paragraph`, `ComposedLine`, `ComposedParagraph`, `BreakpointNode`)
- Measurement abstraction (`Measurer`, `MeasureText`, `GlueSpaceFn`, `GlueSpaceMetrics`, `GetFontMetrics`)
- Layout geometry (`Frame`, `BaselineGrid`)
- Constants (`FORCED_BREAK`, `PROHIBITED`, `HYPHEN_PENALTY`, `DOUBLE_HYPHEN_PENALTY`, `SOFT_HYPHEN_PENALTY`)
- Utility functions (`resolveWeight`)
- Language and alignment types (`Language`, `AlignmentMode`)

Does **not** own:
- Rendering types (owned by `@paragraf/render-core`, `@paragraf/render-pdf`)
- Style resolution types (owned by `@paragraf/style`)
- Template/compile types (owned by `@paragraf/template`, `@paragraf/compile`)
- Any runtime behaviour — this package is types and constants only

## Process Rules

- TDD is mandatory — tests before tasks, no exceptions unless explicitly overridden for a specific workId
- Division of labour: test descriptions are human-authored; test code and task implementations are LLM-generated
- Zero runtime dependencies — this package must never import another `@paragraf/*` package
- No implementation logic except pure type utilities (e.g. `resolveWeight`) — no side effects, no I/O
- Breaking type changes require all dependent packages to be checked: all 11 other packages depend on this
- Layer L0: no layer import rule applies — this is the base layer

---

## Consistency Checks (Copilot-executed)

Pre-task:
- [ ] Active plan workIds exist in `docs/work-pool.md`
- [ ] io-schema.md types match exported TypeScript types in `src/index.ts`
- [ ] dependency.md confirms zero runtime `@paragraf/*` dependencies

Post-task:
- [ ] io-schema.md updated if public types changed
- [ ] io-schema.md changes flagged for sync to project-level `docs/io-schemas.md`
- [ ] If a type used by other packages changed: flag for review of all dependent package io-schema.md files
