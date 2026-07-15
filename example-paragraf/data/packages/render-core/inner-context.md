# render-core — Inner Context

---
type: inner-context
package: @paragraf/render-core
layer: L2
updated: 260416-1950
---

## Role

Composed-paragraph layout engine and multi-target renderer. Converts `ComposedParagraph` output (from `@paragraf/typography`) into positioned segments, then renders those segments to SVG or Canvas. Also owns the document model types (`RenderedItem`, `RenderedPage`, `RenderedDocument`) consumed by `@paragraf/render-pdf`.

---

## Process Rules

- `layoutParagraph` is a pure function: takes `ComposedParagraph + Measurer + origin`, returns `RenderedParagraph`. No side effects.
- LTR layout: words placed left-to-right with `xOffset` (OMA) and `alignOffset` (right/center alignment). Justified lines have `wordSpacing` set by the compositor.
- RTL layout: word order reversed visually (right-to-left). Word widths pre-computed, then placed from `rightEdge` inward.
- `renderToSvg` and `renderToCanvas` share the same glyph-loop logic: get glyphs → GSUB subs → getGlyphPath → advance by `(advanceWidth + kern) * scale + letterSpacing`.
- `metricsCache` is a module-level `Map<fontId, unitsPerEm>` — it is not per-instance. `clearRenderCaches()` must be called in tests to avoid cross-test contamination.
- `getAndSubstituteGlyphs` is exported for consumers (e.g. render-pdf) that need the substituted glyph list without the full render pipeline.
- `BaselineGrid` and `Frame` are re-exported here from `@paragraf/types` for convenience (they were promoted to L0 so `@paragraf/layout` could use them without depending on render-core).

---

## Package Constraints

- Layer L2: may import from L0 (`@paragraf/types`) and L1 (`@paragraf/font-engine`). Never from L2+.
- `metricsCache` is module-level — always call `clearRenderCaches()` in test `afterEach` / `beforeEach`.
- `renderToCanvas` accepts `ctx: any` (browser `CanvasRenderingContext2D`). No type import — avoids a DOM lib dependency.
- `@paragraf/typography` appears only in devDependencies (used in integration tests). Never import it at runtime.

---

## Consistency Checks

- All exported types from `document-types.ts` must also appear in `index.ts` re-exports.
- `RenderedItem.origin` coordinates must be in points (same coordinate space as `Frame`).
- Letter spacing is applied between glyphs but not after the last glyph in a segment.
