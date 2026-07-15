# linebreak — Inner Context

---
type: inner-context
package: @paragraf/linebreak
updated: 260416-1930
---

## Purpose

Package-level consistency and accuracy checker. Consulted at start and end of every inner-loop task for this package.

## Role

Knuth-Plass line-breaking algorithm with hyphenation. Browser-safe — no font I/O, no rendering. Takes a `Paragraph` (node sequence + parameters) and produces `ComposedLine[]` ready for rendering.

Responsible for:
- Knuth-Plass forward pass and optimal breakpoint selection (`computeBreakpoints`)
- Traceback from best `BreakpointNode` to `LineBreak[]` (`traceback`)
- Building the node sequence from hyphenated words and a `Measurer` (`buildNodeSequence`)
- Paragraph composition: node sequence + breakpoints → `ComposedParagraph` (`composeParagraph`)
- Hyphenation: pattern-based per-language (`hyphenateWord`, `hyphenateParagraph`, `loadHyphenator`, `loadLanguages`)
- Hyphenation boundary helpers (`deriveMinLeft`, `deriveMinRight`)
- Test utilities (`mockMeasure`, `mockSpace`, `mockMetrics`)

Does **not** own:
- Font loading or glyph measurement (owned by `@paragraf/font-engine`)
- Style resolution (owned by `@paragraf/style`)
- Frame/page layout (owned by `@paragraf/layout`)
- Paragraph orchestration across fonts and spans (owned by `@paragraf/typography`)

## Process Rules

- TDD is mandatory — tests before tasks, no exceptions unless explicitly overridden for a specific workId
- Division of labour: test descriptions are human-authored; test code and task implementations are LLM-generated
- Browser-safe: no `node:*` imports in production code paths; `loadHyphenator` / `loadLanguages` use dynamic `import()` which is browser-compatible
- Layer L1: may import from `@paragraf/types` (L0) only; never import from L2+

---

## Consistency Checks (Copilot-executed)

Pre-task:
- [ ] Active plan workIds exist in `docs/work-pool.md`
- [ ] io-schema.md types match exported TypeScript types in `src/index.ts`
- [ ] No imports from L2+ packages

Post-task:
- [ ] io-schema.md updated if public types changed
- [ ] io-schema.md changes flagged for sync to project-level `docs/io-schemas.md`
