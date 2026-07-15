# render-core — I/O Schema

---
type: io-schema
package: @paragraf/render-core
updated: 260416-1950
---

## Purpose

All public types, interfaces, classes, and functions exported by `@paragraf/render-core`.

Types consumed but not re-exported: `ComposedParagraph`, `Measurer`, `Font` (from `@paragraf/types`), `FontEngine` (from `@paragraf/font-engine`).

---

## Positioned Output Types

### `PositionedSegment`
```ts
interface PositionedSegment {
  text: string
  font: Font
  x: number   // absolute x on page, in points
  y: number   // absolute y on page (baseline − verticalOffset), in points
}
```

### `RenderedLine`
```ts
interface RenderedLine {
  segments: PositionedSegment[]
  baseline: number      // absolute y of baseline on page, in points
  lineHeight: number    // line height in points
}
```

### `RenderedParagraph`
```ts
type RenderedParagraph = RenderedLine[]
```

---

## Layout Pass

### `layoutParagraph(composed: ComposedParagraph, measurer: Measurer, origin: { x: number; y: number }): RenderedParagraph`
Convert composed paragraph output to positioned segments.

- LTR: places words left-to-right. `line.xOffset` shifts the line (OMA). `alignOffset` shifts for right/center alignment.
- RTL: reverses word order visually. Word widths pre-computed; segments placed right-to-left from `rightEdge`.
- Vertical position: `baseline = lineY + line.baseline`. Segments at `y = baseline − seg.verticalOffset`.
- Advances `lineY` by `line.lineHeight` after each line.

---

## GSUB Helper

### `getAndSubstituteGlyphs(fontEngine: FontEngine, fontId: string, text: string, font?: Font): Glyph[]`
Shape `text` and apply GSUB substitutions:
1. `fontEngine.glyphsForString(fontId, text, font)`
2. `fontEngine.applyLigatures(fontId, glyphs)`
3. If `font.variant === 'superscript'`: `applySingleSubstitution(..., 'sups')`
4. If `font.variant === 'subscript'`: `applySingleSubstitution(..., 'subs')`

Returns the final glyph array. Exported for consumers that need shaped glyphs without rendering.

---

## Renderers

### `renderToSvg(rendered: RenderedParagraph, fontEngine: FontEngine, viewport: { width: number; height: number }): string`
Render to an SVG string.

- Iterates segments → shapes glyphs → calls `getGlyphPath` → collects `toSVG(2)` path strings.
- Advance: `(glyph.advanceWidth + kern) * scale + letterSpacing` (letter spacing not applied after last glyph).
- Returns a full `<svg>` element with `xmlns`, `width`, `height` attributes and one `<path>` element per glyph.

### `renderToCanvas(rendered: RenderedParagraph, fontEngine: FontEngine, ctx: any): void`
Render to a browser `CanvasRenderingContext2D`.

- Same glyph-loop as `renderToSvg`. Issues `ctx.beginPath()` / `ctx.moveTo` / `ctx.lineTo` / `ctx.quadraticCurveTo` / `ctx.bezierCurveTo` / `ctx.closePath` / `ctx.fill()`.
- `ctx` typed as `any` to avoid a DOM lib dependency.

### `clearRenderCaches(): void`
Clear the module-level `metricsCache` (`Map<fontId, unitsPerEm>`). Call in test setup/teardown to avoid cross-test contamination.

---

## Document Model Types

Re-exported from internal `document-types.ts`. `Frame` and `BaselineGrid` are re-exported from `@paragraf/types`.

### `Frame` (re-export from `@paragraf/types`)
```ts
interface Frame {
  page: number
  x: number
  y: number
  width: number
  height: number
  columnCount?: number       // defaults to 1
  gutter?: number            // space between columns in points; defaults to 0
  grid?: BaselineGrid        // when set, line placement snaps to grid
  paragraphSpacing?: number  // vertical gap after each paragraph; defaults to 0
}
```

### `BaselineGrid` (re-export from `@paragraf/types`)
```ts
interface BaselineGrid {
  first: number     // Y-offset from frame.y where the first baseline lands
  interval: number  // distance between baseline grid lines in points
}
```

### `RenderedItem`
```ts
interface RenderedItem {
  origin: { x: number; y: number }
  rendered: RenderedParagraph
}
```
A single rendered paragraph block within a column on a page.

### `RenderedPage`
```ts
interface RenderedPage {
  pageIndex: number
  frame: Frame   // the frame that initiated this page (frame.page === pageIndex)
  items: RenderedItem[]
}
```

### `RenderedDocument`
```ts
interface RenderedDocument {
  pages: RenderedPage[]   // one entry per page that has content
}
```
