# template — I/O Schema

---
type: io-schema
package: @paragraf/template
updated: 260501-0900
---

## Purpose

All public types and functions exported by `@paragraf/template`.

---

## Layout Types

### `Dimension` (re-export from `@paragraf/layout`)
```ts
type Dimension = number | string
```
Number (points) or string with unit: `'20mm'`, `'2cm'`, `'0.5in'`, `'36pt'`, `'100px'`.

### `PageSize` (re-export from `@paragraf/layout`)
```ts
type PageSize = PageSizeName | [number, number]
```
Named size (`'A4'`, `'Letter'`, …) or explicit `[width, height]` tuple in points.

### `DimensionMargins`
```ts
interface DimensionMargins {
  top: Dimension
  right: Dimension
  bottom: Dimension
  left: Dimension
}
```

### `TemplateLayout`
```ts
interface TemplateLayout {
  size: PageSize
  margins: Dimension | DimensionMargins   // single value = equal all sides
  /** @deprecated Use pages with TemplatePageSpec instead. */
  columns?: number
  /** @deprecated Use pages with TemplatePageSpec instead. */
  gutter?: Dimension
  bleed?: Dimension                        // all four sides
  pages?: TemplatePageSpec[]               // per-page region layouts
}
```
`@paragraf/compile` resolves all `Dimension` values to points at compile time.

---

## Region Layout Types

### `TemplateRegionSpec`
```ts
interface TemplateRegionSpec {
  height: Dimension    // required
  columns?: number     // default 1
  gutter?: Dimension   // between columns; default 0
  x?: Dimension        // offset from left of text area; default 0
  y?: Dimension        // offset from top of text area; omit for auto-stack
  width?: Dimension    // default: full text-area width
}
```
Mirrors `RegionSpec` from `@paragraf/layout` but uses `Dimension` strings.

### `TemplatePageSpec`
```ts
interface TemplatePageSpec {
  range: number | string   // see below
  regions: TemplateRegionSpec[]
}
```
`range` values:
- `number` — exact 1-based page number (e.g. `1` = first page only)
- `'N+'` — page N and all subsequent pages (e.g. `'2+'`)
- `'N-M'` — pages N through M inclusive (e.g. `'2-5'`)
- `'default'` — fallback for any page not matched by other entries

Resolution order and conflict handling is delegated to `@paragraf/compile`.

---

## Font Types

### `FontVariantEntry`
```ts
type FontVariantEntry =
  | string                                         // file path shorthand (key looked up in VARIANT_CONVENTIONS)
  | { path: string; weight?: number; style?: FontStyle; stretch?: FontStretch }
```
The four standard keys (`regular`, `bold`, `italic`, `boldItalic`) may use the string shorthand. Custom keys require the object form.

### `TemplateFontVariants`
```ts
interface TemplateFontVariants {
  regular?: FontVariantEntry
  bold?: FontVariantEntry
  italic?: FontVariantEntry
  boldItalic?: FontVariantEntry
  [variant: string]: FontVariantEntry | undefined   // custom weight/style/stretch variants
}
```

### `TemplateFonts`
```ts
type TemplateFonts = Record<string, TemplateFontVariants>
```
Keys are font family names matching those used in style definitions.

---

## Content Types

### `OnMissing`
```ts
type OnMissing = 'skip' | 'placeholder' | 'fallback'
```
- `'skip'` — omit the slot entirely when a binding is missing.
- `'placeholder'` — render a visible placeholder string (handled by compile layer).
- `'fallback'` — render `fallbackText` (must be set when this value is used).

### `ContentSlot`
```ts
interface ContentSlot {
  style: string         // must match a key in Template.styles
  text: string          // literal or with '{{binding.path}}' interpolations
  onMissing?: OnMissing // default 'skip'
  fallbackText?: string // required when onMissing is 'fallback'
}
```

---

## Template

### `Template`
```ts
interface Template {
  layout: TemplateLayout
  fonts: TemplateFonts
  styles: Record<string, ParagraphStyleDef>   // same shape as @paragraf/style defineStyles() input
  content: ContentSlot[]                       // ordered list of content slots
}
```

### `defineTemplate(input: Template): Template`
Validate the template and return it unchanged. Throws on any violation:
- Style inheritance cycles or missing `extends`/`next` references.
- Slot `style` keys that don't exist in `template.styles`.
- `onMissing: 'fallback'` without `fallbackText`.
- Invalid `{{...}}` syntax in any `text` field.

---

## Interpolation

### `Token`
```ts
type Token =
  | { type: 'literal'; value: string }
  | { type: 'binding'; path: string }
```

### `parseTokens(text: string): Token[]`
Parse a content slot's `text` string into tokens.
- Plain strings → single `literal` token.
- `'{{product.name}}'` → single `binding` token.
- `'Article: {{sku}}'` → `[literal, binding]`.
- Multiple `{{...}}` per string are supported.

Throws if:
- `{{` has no matching `}}`
- Path is empty (`{{}}`)
- Path contains invalid segments (e.g. starts with a digit, contains spaces)

Valid path examples: `'name'`, `'product.sku'`, `'items.0.price'`.

---

## Re-exports

### `ParagraphStyleDef` (re-export from `@paragraf/style`)
The input shape for a paragraph style. Re-exported so consumers can type `Template.styles` without a separate `@paragraf/style` import.
