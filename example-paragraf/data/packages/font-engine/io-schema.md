# font-engine — I/O Schema

---
type: io-schema
package: @paragraf/font-engine
updated: 260416-1800
---

## Purpose

Public types and functions exported by this package.
Subset of project-level `docs/io-schemas.md` scoped to `@paragraf/font-engine` only.

---

## Exported Interfaces

### `PathCommand`
```ts
interface PathCommand {
  command: 'moveTo' | 'lineTo' | 'quadraticCurveTo' | 'bezierCurveTo' | 'closePath';
  args: number[];
}
```
A single SVG-style path drawing command.

### `GlyphPath`
```ts
interface GlyphPath {
  commands: PathCommand[];
  toSVG(precision?: number): string;
  boundingBox?: { minX: number; minY: number; maxX: number; maxY: number };
}
```
Vector path for a glyph outline.

### `Glyph`
```ts
interface Glyph {
  index: number;           // glyph index in the font
  advanceWidth: number;
  xOffset?: number;        // font units; GPOS positional x adjustment
  yOffset?: number;        // font units; GPOS positional y adjustment
  codePoints?: number[];
}
```
Opaque handle to a shaped glyph. Engine-agnostic.

### `FontEngine`
```ts
interface FontEngine {
  loadFont(id: string, path: string): Promise<void>;
  glyphsForString(fontId: string, text: string, font?: Font): Glyph[];
  glyphForCodePoint?(fontId: string, codePoint: number): Glyph | null;
  /** @deprecated GSUB is applied inside glyphsForString. Optional — custom engines need not implement this. */
  applyLigatures?(fontId: string, glyphs: Glyph[]): Glyph[];
  /** @deprecated GSUB is applied inside glyphsForString. Optional — custom engines need not implement this. */
  applySingleSubstitution?(fontId: string, glyphs: Glyph[], featureTag: 'sups' | 'subs'): Glyph[];
  getKerning(fontId: string, glyph1: Glyph, glyph2: Glyph): number;
  getGlyphPath(fontId: string, glyph: Glyph, x: number, y: number, fontSize: number): GlyphPath;
  getFontMetrics(fontId: string, fontSize: number, variant?: 'normal' | 'superscript' | 'subscript'): FontMetrics;
}
```
Pluggable abstraction for font access. Implementations are interchangeable.

---

## Exported Classes

### `FontkitEngine`
Implements `FontEngine` using the `fontkit` v2 library.

---

## Exported Functions

### `createMeasurer(registry, measure?, space?, metrics?) → Measurer`
```ts
createMeasurer(
  registry: FontRegistry,
  measure?: MeasureText,
  space?: GlueSpaceFn,
  metrics?: GetFontMetrics,
): Measurer
```
Factory that builds a `Measurer` from a `FontRegistry`. The optional `measure`, `space`, and `metrics` overrides replace the default fontkit-backed implementations — used in tests to inject mocks.

### `loadFontkitFont(filePath, fontId) → fontkitFont`
Loads a font file via fontkit. Caches by fontId. Exported for shared use by other modules.

### `resolveFontkitFont(font, registry) → fontkitFont`
Resolves a `Font` descriptor from a `FontRegistry` and calls `loadFontkitFont`.

---

## Exported Testing Utilities

### `mockMeasure: MeasureText`
Deterministic mock: `content.length * font.size * 0.6 + letterSpacing`.

### `mockSpace: GlueSpaceFn`
Deterministic mock: `{ width: size*0.25, stretch: size*0.125, shrink: size*0.075 }`.

### `mockMetrics: GetFontMetrics`
Deterministic mock with fixed proportional metrics. Supports superscript/subscript variant baseline shifts.

---

## Types Imported from `@paragraf/types` (not re-exported)

Used internally or passed through — not owned by this package:

| Type | Purpose |
|---|---|
| `Font` | Font descriptor (id, size, variant, letterSpacing) |
| `FontMetrics` | ascender, descender, xHeight, capHeight, lineGap, baselineShift |
| `FontRegistry` | Map of font id → `{ filePath }` |
| `Measurer` | Combined `{ measure, space, metrics, registry }` — output of `createMeasurer` |
| `MeasureText` | `(content: string, font: Font) => number` |
| `GlueSpaceFn` | `(font: Font) => GlueSpaceMetrics` |
| `GlueSpaceMetrics` | `{ width, stretch, shrink }` |
| `GetFontMetrics` | `(font: Font) => FontMetrics` |
