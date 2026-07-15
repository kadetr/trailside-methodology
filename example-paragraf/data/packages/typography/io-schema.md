# typography — I/O Schema

---
type: io-schema
package: @paragraf/typography
updated: 260416-1955
---

## Purpose

All public types, interfaces, classes, and functions exported by `@paragraf/typography`.

Types consumed but not re-exported: `ComposedParagraph`, `TextSpan`, `SpanSegment`, `Font`, `FontRegistry`, `AlignmentMode`, `Language`, `Measurer`, `GlueSpaceMetrics` (all from `@paragraf/types`).

---

## Paragraph Composition

### `ParagraphInput`
```ts
interface ParagraphInput {
  text?: string                              // plain text (single font)
  font?: Font                                // optional when spans provided; required for text mode (throws if absent)
  fontPerWord?: (index: number, word: string) => Font  // per-word font override (text path only)
  spans?: TextSpan[]                         // rich input; mutually exclusive with text (spans win)
  lineWidth: number                          // column width in points
  lineWidths?: number[]                      // per-line width overrides
  tolerance?: number                         // default 2
  emergencyStretch?: number                  // default 0
  firstLineIndent?: number                   // default 0
  alignment?: AlignmentMode                  // default 'justified'
  language?: Language                        // default 'en-us'
  looseness?: number                         // default 0 (clamped to integer for WASM)
  justifyLastLine?: boolean                  // default false
  consecutiveHyphenLimit?: number            // default 0 (no limit)
  /** Demerit added when the final line contains a single word. @since v0.6 */
  runtPenalty?: number                       // default 0
  /** Demerit added when the entire paragraph fits on a single line. @since v0.6 */
  singleLinePenalty?: number                 // default 0
  /** @deprecated Use `runtPenalty` instead. */
  widowPenalty?: number                      // default 0
  /** @deprecated Use `singleLinePenalty` instead. */
  orphanPenalty?: number                     // default 0
  preserveSoftHyphens?: boolean              // default true
  lineHeight?: number                        // when set, overrides font-metric-derived leading
  opticalMarginAlignment?: boolean           // when true, triggers two-pass OMA recompose
}
```

### `ParagraphOutput`
```ts
interface ParagraphOutput {
  lines: ComposedParagraph
  lineCount: number
  usedEmergency: boolean
}
```

### `ParagraphComposer`
```ts
interface ParagraphComposer {
  compose(input: ParagraphInput): ParagraphOutput
  ensureLanguage(language: Language): Promise<void>
  measurer?: Measurer    // the internal Measurer; pass to layoutDocument to avoid a second cache
}
```

### `ComposerOptions`
```ts
interface ComposerOptions {
  useWasm?: boolean              // default auto-detect; false forces TS path
  measureCache?: MeasureCacheOptions
}
```

### `createParagraphComposer(registry: FontRegistry, options?: ComposerOptions): Promise<ParagraphComposer>`
Primary factory. Async only for initial setup (loads `en-us` hyphenation patterns).
All subsequent `compose()` calls are synchronous.

### `createDefaultFontEngine(registry: FontRegistry, options?: ComposerOptions): Promise<FontEngine>`
Create a `FontEngine` (WASM or fontkit) consistent with the composer's backend.
Use when the caller needs a `FontEngine` for rendering (`renderToSvg` / `renderToPdf`).

### `wasmStatus(): { status: 'loaded' | 'absent' | 'error'; error?: string }`
Diagnostic: returns current WASM loading status.
- `'loaded'` — WASM active.
- `'absent'` — package not found (wasm-pack not run).
- `'error'` — package found but failed to init; `error` contains the message.

---

## Measurement Cache

### `FeatureConfig`
```ts
type FeatureConfig = Record<string, boolean>
```
A record of GSUB feature flags keyed by OpenType feature tag (e.g. `'liga'`, `'calt'`).
Pass to `featureSetIdFromConfig` to derive a deterministic cache-key segment.

### `featureSetIdFromConfig(config: FeatureConfig): string`
Derives a deterministic, stable string from a `FeatureConfig`.
Keys are sorted before serialization — `{ liga: true, calt: false }` and
`{ calt: false, liga: true }` produce the same ID.
Pass the result as `MeasureCacheOptions.featureSetId` to guarantee cache consistency
across composer instances and process restarts.

### `MeasureCacheOptions`
```ts
interface MeasureCacheOptions {
  enabled?: boolean                                        // default true
  maxCacheEntries?: number                                 // default 10_000
  featureSetId?: string                                    // static GSUB feature group label
  featureSetIdResolver?: (word: string, font: Font) => string  // dynamic resolver; wins over static+featureConfig
  featureConfig?: FeatureConfig                            // derive ID via featureSetIdFromConfig; wins over featureSetId
  registryId?: string                                      // scope cache to this font registry instance (#76)
}
```
Precedence for featureSetId resolution: `featureSetIdResolver` → `featureConfig` → `featureSetId` → `'__default-feature-set__'`

### `MeasureCacheStats`
```ts
interface MeasureCacheStats {
  size: number
  hits: number
  misses: number
  evictions: number
}
```

### `getMeasureCacheStats(): MeasureCacheStats`
Snapshot of the module-level measurement LRU cache stats.

### `clearMeasureCache(): void`
Clear the module-level cache and reset all stats to zero.

### `configureMeasureCache(options?: MeasureCacheOptions): { enabled: boolean; maxCacheEntries: number }`
Update module-level defaults for new composers. Returns the effective config.

---

## Optical Margin Alignment

### `PROTRUSION_TABLE: Map<string, { left: number; right: number }>`
Protrusion fractions per character. Values are fractions of the character's own advance width (pdfTeX-style). Covers: hyphens, dashes, sentence-ending punctuation, curly/straight quotes, brackets, asterisk.

### `lookupProtrusion(char: string): { left: number; right: number }`
Return protrusion fractions for a single character. Returns `{ left: 0, right: 0 }` for unknown or empty input.

### `buildOmaAdjustments(lines: ComposedParagraph, baseWidth: number, measurer?: Measurer): { lineWidths: number[]; xOffsets: number[]; rightProtrusions: number[] }`
Compute per-line protrusion adjustments. For each line: inspect first/last character, scale by advance width (or `font.size` if no measurer). Returns adjusted `lineWidths`, negative `xOffsets` (left hang), and `rightProtrusions`.

### `buildOmaInput(input: ParagraphInput, firstPassLines: ComposedParagraph, measurer?: Measurer): ParagraphInput`
Build the pass-2 input from a pass-1 result. Fills `lineWidths` with OMA-adjusted values; sets `opticalMarginAlignment: false` to prevent infinite recursion.

---

## Document Model

### `Document`
```ts
interface Document {
  paragraphs: ParagraphInput[]
  frames: Frame[]
  styleDefaults?: Partial<ParagraphInput>   // merged into each paragraph; per-para fields win
}
```

### `ComposedDocument`
```ts
interface ComposedDocument {
  paragraphs: Array<{ input: ParagraphInput; output: ParagraphOutput }>
}
```

### `snapCursorToGrid(cursorY: number, baseline: number, frame: Frame, grid: BaselineGrid): number`
Snap `cursorY` so `cursorY + baseline` lands on the next baseline grid line at or above its current position.

### `gridAdvance(lineHeight: number, interval: number): number`
Round `lineHeight` up to the nearest multiple of `interval`. The amount to advance `cursorY` after placing a grid-snapped line.

### `deriveLineWidths(paragraphs: ParagraphInput[], frames: Frame[], frameAssignments?: number[]): ParagraphInput[]`
Return a copy of `paragraphs` with `lineWidth` set to the column width of each paragraph's assigned frame. Use before `composeDocument` when frames have different column widths. `frameAssignments[i]` defaults to 0.

### `composeDocument(doc: Document, composer: ParagraphComposer): ComposedDocument`
Phase 1. Merges `styleDefaults` into each paragraph (per-para fields win), then overrides `lineWidth` with `colWidth(frames[0])`. Calls `composer.compose()` for each paragraph. Warns (console) when frames have differing column widths.

### `layoutDocument(composed: ComposedDocument, frames: Frame[], measurer: Measurer): RenderedDocument`
Phase 2. Places composed lines into frames, columns, and pages. Text flows top-to-bottom, column-by-column, frame-by-frame. A paragraph may split across columns/frames. Returns `RenderedDocument`.

---

## Re-exported Document Types

These are pulled from `@paragraf/render-core` (and transitively from `@paragraf/types`) and re-exported so consumers need not add render-core directly.

| Type | Origin |
|---|---|
| `BaselineGrid` | `@paragraf/types` via render-core |
| `Frame` | `@paragraf/types` via render-core |
| `RenderedItem` | `@paragraf/render-core` |
| `RenderedPage` | `@paragraf/render-core` |
| `RenderedDocument` | `@paragraf/render-core` |
