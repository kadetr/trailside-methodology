# compile — I/O Schema

---
type: io-schema
package: @paragraf/compile
updated: 260418-1800
---

## Purpose

All public types and functions **owned by** `@paragraf/compile`. Re-exported lower-layer APIs are listed separately at the end.

---

## Main Entry Points

### `OutputFormat`
```ts
type OutputFormat = 'pdf' | 'svg' | 'rendered'
```

### `OverflowBehavior`
```ts
type OverflowBehavior = 'silent' | 'throw'
```

### `ShapingMode`
```ts
type ShapingMode = 'auto' | 'wasm' | 'fontkit'
```

### `CompileOptions<T = unknown>`
```ts
interface CompileOptions<T> {
  template: Template                           // validated template from defineTemplate()
  data: T                                      // data record to interpolate into slots
  normalize?: (raw: T) => Record<string, unknown>  // optional data normalizer
  output?: OutputFormat                        // default 'pdf'
  basePath?: string                            // base path for font resolution; default process.cwd()
  onOverflow?: OverflowBehavior                // default 'silent'
  shaping?: ShapingMode                        // default 'auto'
  title?: string                               // PDF Info dict title
  lang?: string                                // PDF Info dict language tag
  selectable?: boolean                         // invisible text layer; pdf only; default false
  outputIntent?: OutputIntent                  // ICC OutputIntent for PDF/A or PDF/X; pdf only; default undefined
  maxPages?: number                            // default 100; throws RangeError if < 1
}
```

**Color pipeline (internal, pdf output only):** when `outputIntent` is provided, `compile()` calls `loadColorWasm()` + `loadBuiltinSrgb()` + `createWasmTransform()` to build a `ColorTransform` and passes it to `renderDocumentToPdf`. No color-wasm APIs are re-exported.

### `CompileResult`
```ts
interface CompileResult {
  data: Buffer | string | RenderedDocument   // Buffer=pdf, string=svg, RenderedDocument=rendered
  metadata: {
    pageCount: number
    overflowLines: number      // lines that did not fit within maxPages
    shapingEngine: 'wasm' | 'fontkit'
  }
}
```

### `compile<T = unknown>(options: CompileOptions<T>): Promise<CompileResult>`
Compile a single document through the 11-step pipeline. Returns `CompileResult`.

---

## Batch Compilation

### `CompileBatchOptions<T>`
```ts
interface CompileBatchOptions<T> extends Omit<CompileOptions<T>, 'data'> {
  records: T[]
  concurrency?: number                          // default 4; throws RangeError if < 1
  onProgress?: (completed: number, total: number) => void
}
```

### `CompileBatchResult<T>`
```ts
interface CompileBatchResult<T> {
  record: T
  index: number         // 0-based index into records[]
  result?: CompileResult
  error?: Error
}
```
Exactly one of `result` or `error` is set per entry.

### `compileBatch<T>(options: CompileBatchOptions<T>): Promise<CompileBatchResult<T>[]>`
Compile multiple records with bounded concurrency. Returns one result per record, in original order.

---

## Font Utilities

### `VARIANT_CONVENTIONS: Record<string, { weight: number; style: FontStyle }>`
18-key convention table mapping variant key names to weight and style metadata.
Keys: `thin`, `extraLight`, `light`, `regular`, `medium`, `semiBold`, `bold`, `extraBold`, `black` + italic variants of each.

### `resolveVariantEntry(key: string, entry: FontVariantEntry, basePath: string): ResolvedVariant`
Expand a `FontVariantEntry` to its full metadata (path, weight, style, stretch). Emits console.warn for unknown shorthand keys.

### `selectVariant(family: string, variants: ResolvedVariant[], weight: number, style: FontStyle): ResolvedVariant`
CSS nearest-weight variant selection for a given family + weight + style query.

### `buildFontRegistry(fonts: TemplateFonts, basePath: string): FontRegistry`
Resolve all font file paths in a `TemplateFonts` map against `basePath`. Throws if any file does not exist.

---

## Interpolation

### `resolveText(text: string, data: Record<string, unknown>): string | null`
Resolve a content slot's `text` string against a data record. Dot-path traversal for binding tokens. Returns `null` if any binding resolves to `null` / `undefined` (all-or-nothing).

---

## Re-exported Lower-Layer APIs

`@paragraf/compile` re-exports the following for consumers who depend only on this package:

| Symbol | Origin |
|---|---|
| `Font`, `FontId`, `FontMetrics`, `FontRegistry`, `FontDescriptor`, `Language`, `AlignmentMode`, `ComposedLine`, `ComposedParagraph`, `Measurer`, `GlueSpaceMetrics`, `MeasureText`, `TextSpan`, `SpanSegment` | `@paragraf/types` |
| `FontEngine`, `Glyph`, `GlyphPath`, `PathCommand` | `@paragraf/font-engine` |
| `computeBreakpoints`, `traceback`, `buildNodeSequence`, `composeParagraph`, `loadHyphenator`, `loadLanguages`, `hyphenateParagraph`, `hyphenateWord`, `DEFAULT_HYPHENATE_OPTIONS`, `HyphenatedWordWithFont`, `HyphenateOptions`, `HyphenatedWord` | `@paragraf/linebreak` |
| `createParagraphComposer`, `createDefaultFontEngine`, `buildOmaAdjustments`, `buildOmaInput`, `lookupProtrusion`, `ParagraphInput`, `ParagraphOutput`, `ParagraphComposer`, `ComposerOptions` | `@paragraf/typography` |
| `layoutParagraph`, `renderToSvg` | `@paragraf/render-core` |
| `Template` (type) | `@paragraf/template` |
