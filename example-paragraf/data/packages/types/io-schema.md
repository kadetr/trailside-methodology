# types — I/O Schema

---
type: io-schema
package: @paragraf/types
updated: 260416-1830
---

## Purpose

All public types, interfaces, and constants exported by this package.
This is the project-level source of truth — all other package io-schema.md files
that reference these types are subsets of this file.

---

## Font Types

### `FontId`
```ts
type FontId = string
```

### `FontStyle`
```ts
type FontStyle = 'normal' | 'italic' | 'oblique'
```

### `FontStretch`
```ts
type FontStretch = 'condensed' | 'semi-condensed' | 'normal' | 'semi-expanded' | 'expanded'
```

### `FontWeight`
```ts
type FontWeight = number | 'thin' | 'extra-light' | 'light' | 'normal' | 'medium' | 'semi-bold' | 'bold' | 'extra-bold' | 'black'
```
Authoring-only. Must be resolved to `number` via `resolveWeight()` before passing to engine types.

### `FontVariant`
```ts
type FontVariant = 'normal' | 'superscript' | 'subscript'
```

### `Font`
```ts
interface Font {
  id: FontId
  size: number
  weight: number          // always numeric — use resolveWeight() on FontSpec.weight
  style: FontStyle
  stretch: FontStretch
  letterSpacing?: number  // extra space between chars, same unit as size; default 0
  variant?: FontVariant   // triggers GSUB sups/subs; default 'normal'
}
```

### `FontSpec`
```ts
interface FontSpec {
  family?: string
  size?: number
  weight?: FontWeight     // named or numeric; resolved at style layer
  style?: FontStyle
  stretch?: FontStretch
  letterSpacing?: number
  variant?: FontVariant
}
```
Authoring-time font description. All fields optional for partial inheritance.

### `FontDescriptor`
```ts
interface FontDescriptor {
  id: FontId
  family: string
  filePath: string
  weight?: number
  style?: FontStyle
  stretch?: FontStretch
}
```

### `FontRegistry`
```ts
type FontRegistry = Map<FontId, FontDescriptor>
```

### `FontMetrics`
```ts
interface FontMetrics {
  unitsPerEm: number
  ascender: number        // sTypoAscender scaled to font.size
  descender: number       // sTypoDescender scaled to font.size (negative)
  xHeight: number
  capHeight: number
  lineGap: number
  baselineShift: number   // positive = raise (sups), negative = lower (subs), 0 = normal
}
```

---

## Span Types

### `TextSpan`
```ts
interface TextSpan {
  text: string
  font: Font
  verticalOffset?: number   // positive = above baseline, negative = below
}
```
Single-font text run. May contain whitespace (word boundaries).

### `SpanSegment`
```ts
interface SpanSegment {
  text: string
  font: Font
  verticalOffset?: number   // propagated from source TextSpan
}
```
Single-font run within a word — no whitespace.

---

## Language and Alignment

### `Language`
```ts
type Language = 'en-us' | 'en-gb' | 'de' | 'fr' | 'tr' | 'nl' | 'pl' | 'it' | 'es' | 'sv' | 'no' | 'da' | 'fi' | 'hu' | 'cs' | 'sk' | 'ro' | 'hr' | 'sl' | 'lt' | 'lv' | 'et'
```

### `AlignmentMode`
```ts
type AlignmentMode = 'justified' | 'left' | 'right' | 'center'
```

---

## Line-Breaking Nodes

### `Box`
```ts
interface Box {
  type: 'box'
  width: number
  content: string
  font: Font
  verticalOffset?: number
}
```

### `Glue`
```ts
interface Glue {
  type: 'glue'
  kind: 'word' | 'termination'
  width: number
  stretch: number
  shrink: number
  font?: Font
}
```

### `Penalty`
```ts
interface Penalty {
  type: 'penalty'
  width: number
  penalty: number
  flagged: boolean
}
```

### `Node`
```ts
type Node = Box | Glue | Penalty
```

### `BreakpointNode`
```ts
interface BreakpointNode {
  position: number
  line: number
  totalDemerits: number
  ratio: number
  previous: BreakpointNode | null
  flagged: boolean
  consecutiveHyphens: number
}
```

---

## Paragraph I/O

### `Paragraph`
```ts
interface Paragraph {
  nodes: Node[]
  lineWidth: number
  lineWidths?: number[]           // per-line widths for multi-column
  tolerance: number
  emergencyStretch?: number
  firstLineIndent?: number
  alignment?: AlignmentMode
  looseness?: number
  justifyLastLine?: boolean
  consecutiveHyphenLimit?: number
  /** Demerit added when the final line contains a single word. @since v0.6 */
  runtPenalty?: number
  /** Demerit added when the entire paragraph fits on a single line. @since v0.6 */
  singleLinePenalty?: number
  /** @deprecated Use `runtPenalty` instead. Will be removed in v1.0. */
  widowPenalty?: number
  /** @deprecated Use `singleLinePenalty` instead. Will be removed in v1.0. */
  orphanPenalty?: number
}
```

### `ComposedLine`
```ts
interface ComposedLine {
  words: string[]
  fonts: Font[]
  wordRuns: SpanSegment[][]       // per-word span detail
  wordSpacing: number
  hyphenated: boolean
  ratio: number
  alignment: AlignmentMode
  isWidow: boolean
  lineWidth: number
  lineHeight: number
  baseline: number
  direction?: 'ltr' | 'rtl'
  xOffset?: number                // left shift for Optical Margin Alignment
  rightProtrusion?: number        // right overhang for OMA
}
```

### `ComposedParagraph`
```ts
type ComposedParagraph = ComposedLine[]
```

---

## Measurement Abstraction

### `MeasureText`
```ts
type MeasureText = (content: string, font: Font) => number
```

### `GlueSpaceMetrics`
```ts
interface GlueSpaceMetrics {
  width: number
  stretch: number
  shrink: number
}
```

### `GlueSpaceFn`
```ts
type GlueSpaceFn = (font: Font) => GlueSpaceMetrics
```

### `GetFontMetrics`
```ts
type GetFontMetrics = (font: Font) => FontMetrics
```

### `Measurer`
```ts
interface Measurer {
  measure: MeasureText
  space: GlueSpaceFn
  metrics: GetFontMetrics
  registry: FontRegistry
}
```

---

## Layout Geometry

### `BaselineGrid`
```ts
interface BaselineGrid {
  first: number     // Y-offset from frame.y where first baseline lands
  interval: number  // distance between baseline grid lines in points
}
```

### `Frame`
```ts
interface Frame {
  page: number
  x: number
  y: number
  width: number
  height: number
  columnCount?: number
  gutter?: number
  grid?: BaselineGrid
  paragraphSpacing?: number
}
```

---

## Constants

| Constant | Value | Meaning |
|---|---|---|
| `FORCED_BREAK` | `-Infinity` | Penalty forcing a line break |
| `PROHIBITED` | `+Infinity` | Penalty prohibiting a line break |
| `HYPHEN_PENALTY` | `50` | Standard hyphen penalty |
| `DOUBLE_HYPHEN_PENALTY` | `3000` | Penalty for consecutive hyphenated lines |
| `SOFT_HYPHEN_PENALTY` | `0` | Soft hyphen (U+00AD) penalty |

---

## Utility Functions

### `resolveWeight(w: FontWeight): number`
Resolves a `FontWeight` authoring value to its numeric equivalent. Numeric values pass through unchanged.
