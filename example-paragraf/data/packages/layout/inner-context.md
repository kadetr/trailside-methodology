# layout — Inner Context

---
type: inner-context
package: @paragraf/layout
updated: 260501-0900
---

## Purpose

Package-level consistency and accuracy checker. Consulted at start and end of every inner-loop task for this package.

## Role

Page geometry, unit converters, and named page sizes. Pure geometry — no ink, no colour, no rendering. Produces `Frame[]` arrays consumed by `@paragraf/typography` for text composition.

Responsible for:
- Unit converters: `mm`, `cm`, `inch`, `px`, `parseDimension` (all return points)
- `Dimension` type: `number | string` with unit suffix (`'20mm'`, `'2cm'`, `'0.5in'`, `'36pt'`, `'100px'`)
- Named page sizes: ISO A/B series, SRA3/4, North American Letter/Legal/Tabloid (`PAGE_SIZES`, `PageSizeName`, `PageSize`)
- Orientation helpers: `landscape`, `portrait`, `resolvePageSize`
- Page layout: `PageLayout` class → `frames(pageCount)` producing `Frame[]` with bleed, margins, multi-column geometry
- `columnWidths(frame)` utility — distributes frame width across columns accounting for gutter
- **Region layout**: `RegionSpec` interface and `framesForRegions(regions, textX, textY, textWidth, page)` — compose arbitrary rectangular regions into `Frame[]` with auto-stack and per-region column splitting

Does **not** own:
- Text frame content or text flow (owned by `@paragraf/typography`)
- Baseline grid computation (defined in `@paragraf/types` as `BaselineGrid`; applied in `@paragraf/typography`)
- Multi-column text flow across frames (owned by `@paragraf/compile` which calls `framesForRegions`)
- Rendering or PDF page setup (owned by `@paragraf/render-pdf`)

## Process Rules

- TDD is mandatory — tests before tasks, no exceptions unless explicitly overridden for a specific workId
- Division of labour: test descriptions are human-authored; test code and task implementations are LLM-generated
- Layer L1: may import from `@paragraf/types` (L0) only; never import from L2+
- All geometry in points (pt). Unit converters at the boundary — callers use `mm()`, `inch()` etc. before passing values
- `PageLayout` validates all inputs eagerly at construction time (throws on invalid bleed, columns, margins)

---

## Consistency Checks (Copilot-executed)

Pre-task:
- [ ] Active plan workIds exist in `docs/work-pool.md`
- [ ] io-schema.md types match exported TypeScript types in `src/index.ts`
- [ ] No imports from L2+ packages

Post-task:
- [ ] io-schema.md updated if public types changed
- [ ] io-schema.md changes flagged for sync to project-level `docs/io-schemas.md`
