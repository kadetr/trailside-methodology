# style — I/O Schema

---
type: io-schema
package: @paragraf/style
updated: 260430-1800
---

## Purpose

All public types, interfaces, classes, and functions exported by `@paragraf/style`.

Types re-exported from `@paragraf/types` (for consumer convenience — no separate import needed):
`FontWeight`, `FontStyle`, `FontStretch`, `FontVariant`, `resolveWeight`, `FontSpec`.

---

## Font Feature Types

### `FontFeatures`
```ts
type FontFeatures = Record<string, boolean>;
```
OpenType feature config. Keys are feature tags (e.g. `'liga'`, `'kern'`), values are `true`/`false`.

### `featureSetIdFromConfig(config: FontFeatures): string`
Produces a deterministic, insertion-order-independent string key from a `FontFeatures` map.
Sorts keys alphabetically, then `JSON.stringify`s the `[key, value]` pairs.
Used to derive a stable cache key for font shaping lookups.

---

## Paragraph Style Types

### `ParagraphStyleDef`
```ts
interface ParagraphStyleDef {
  extends?: string          // name of parent style in the same registry

  // Typography
  font?: FontSpec           // merged field-by-field with parent
  language?: Language
  alignment?: AlignmentMode
  lineHeight?: number       // total line height in points (leading)
  hyphenation?: boolean     // default true

  // Spacing
  spaceBefore?: number      // vertical space above paragraph in points
  spaceAfter?: number       // vertical space below paragraph in points
  firstLineIndent?: number  // first-line indent in points

  // KP algorithm tuning
  tolerance?: number        // KP tolerance; default 2
  looseness?: number        // KP looseness; default 0

  // Style flow
  next?: string             // name of style to apply to the following paragraph

  // OpenType features
  features?: FontFeatures   // feature config for cache-key derivation via featureSetIdFromConfig
}
```
Authoring-time definition. All fields optional — unset fields inherit from parent or defaults.

### `ResolvedParagraphStyle`
```ts
interface ResolvedParagraphStyle {
  font: Required<FontSpec>  // all font fields guaranteed present after resolution
  language: Language
  alignment: AlignmentMode
  lineHeight: number
  hyphenation: boolean
  spaceBefore: number
  spaceAfter: number
  firstLineIndent: number
  tolerance: number
  looseness: number
  next?: string             // only present if declared in the inheritance chain
  features?: FontFeatures   // only present if declared in the inheritance chain
}
```
Fully-merged flat output. Safe to pass directly to `@paragraf/typography`.

**Built-in defaults** (applied when no style chain sets a value):

| Field | Default |
|---|---|
| `font.size` | `10` |
| `font.weight` | `400` |
| `font.style` | `'normal'` |
| `font.stretch` | `'normal'` |
| `font.letterSpacing` | `0` |
| `font.variant` | `'normal'` |
| `language` | `'en-us'` |
| `alignment` | `'justified'` |
| `lineHeight` | `14` |
| `hyphenation` | `true` |
| `spaceBefore` | `0` |
| `spaceAfter` | `0` |
| `firstLineIndent` | `0` |
| `tolerance` | `2` |
| `looseness` | `0` |
| `features` | `undefined` |

### `StyleRegistry`
```ts
class StyleRegistry {
  has(name: string): boolean
  resolve(name: string): ResolvedParagraphStyle   // throws if name not defined
  get(name: string): ParagraphStyleDef | undefined
  names(): string[]
}
```
Resolved styles are memoized per name. Inheritance chains are walked from root to leaf (root values applied first, leaf values override).

### `defineStyles(defs: Record<string, ParagraphStyleDef>): StyleRegistry`
Validate and create a `StyleRegistry`. Throws on:
- `extends` pointing to an undefined style name
- `next` pointing to an undefined style name
- Circular inheritance chains

---

## Character Style Types

### `CharStyleDef`
```ts
interface CharStyleDef {
  font?: Partial<FontSpec>  // font.letterSpacing is the authoritative tracking override
  color?: string            // CSS hex/rgb string — stored, not rendered here
}
```

### `ResolvedCharStyle`
```ts
interface ResolvedCharStyle {
  font: Partial<FontSpec>
  color?: string
}
```

### `CharStyleRegistry`
```ts
class CharStyleRegistry {
  has(name: string): boolean
  resolve(name: string): ResolvedCharStyle   // throws if name not defined
  get(name: string): CharStyleDef | undefined
  names(): string[]
}
```
No inheritance — character styles are flat. No caching needed.

### `defineCharStyles(defs: Record<string, CharStyleDef>): CharStyleRegistry`
Create a `CharStyleRegistry`. No validation beyond existence checks at resolve time.

---

## Re-exports from `@paragraf/types`

| Export | Notes |
|---|---|
| `FontSpec` | Authoring-time font description |
| `FontWeight` | Named or numeric weight |
| `FontStyle` | `'normal' \| 'italic' \| 'oblique'` |
| `FontStretch` | `'condensed' \| … \| 'expanded'` |
| `FontVariant` | `'normal' \| 'superscript' \| 'subscript'` |
| `resolveWeight` | Resolve named weight to number |
