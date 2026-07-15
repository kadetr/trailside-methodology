# style — Inner Context

---
type: inner-context
package: @paragraf/style
updated: 260430-1800
---

## Purpose

Package-level consistency and accuracy checker. Consulted at start and end of every inner-loop task for this package.

## Role

Paragraph and character style definitions with inheritance resolution. Provides a typed style registry that resolves named styles (with `extends` chains) to fully-merged flat structs ready for the typography layer.

Responsible for:
- Paragraph style definition and resolution (`ParagraphStyleDef`, `ResolvedParagraphStyle`, `StyleRegistry`, `defineStyles`)
- Character style definition and resolution (`CharStyleDef`, `ResolvedCharStyle`, `CharStyleRegistry`, `defineCharStyles`)
- Re-exporting font descriptor helpers for consumer convenience (`FontWeight`, `FontStyle`, `FontStretch`, `FontVariant`, `resolveWeight`)
- Font feature config: `FontFeatures` type, `featureSetIdFromConfig` deterministic hash helper, `features` field propagation through style inheritance
- Inheritance validation: circular chain detection, unresolved `extends`/`next` references
- Built-in defaults: font size 10pt, `en-us`, `justified`, 14pt line height, tolerance 2

Does **not** own:
- Font loading or measurement (owned by `@paragraf/font-engine`)
- Rendering or PDF output
- Paragraph composition (owned by `@paragraf/typography`)

## Process Rules

- TDD is mandatory — tests before tasks, no exceptions unless explicitly overridden for a specific workId
- Division of labour: test descriptions are human-authored; test code and task implementations are LLM-generated
- Layer L1: may import from `@paragraf/types` (L0) only; never import from L2+
- `ResolvedParagraphStyle.font` is `Required<FontSpec>` — all font fields are guaranteed present after resolution
- `CharStyleDef.font` uses `Partial<FontSpec>` — `letterSpacing` is the authoritative tracking override

---

## Consistency Checks (Copilot-executed)

Pre-task:
- [ ] Active plan workIds exist in `docs/work-pool.md`
- [ ] io-schema.md types match exported TypeScript types in `src/index.ts`
- [ ] No imports from L2+ packages

Post-task:
- [ ] io-schema.md updated if public types changed
- [ ] io-schema.md changes flagged for sync to project-level `docs/io-schemas.md`
