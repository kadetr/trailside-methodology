# shaping-wasm — I/O Schema

---
type: io-schema
package: @paragraf/shaping-wasm
updated: 260416-1945
---

## Purpose

All public types, interfaces, classes, and functions exported by `@paragraf/shaping-wasm`.

Re-exports from `@paragraf/font-engine` (consumed but not re-exported): `FontEngine`, `Glyph`, `GlyphPath`, `PathCommand`.

---

## WASM Loader

### `loadShapingWasm(): unknown`
Load the compiled Rust/WASM module synchronously via `createRequire`.
Returns the raw wasm-bindgen JS module. Throws if `wasm/pkg/knuth_plass_wasm.js` is not present (i.e. `wasm-pack` has not been run).

Must be called once before constructing `WasmFontEngine` with the face API. Pass the return value as the first argument to `new WasmFontEngine(wasm, opts)`.

---

## Face Cache Stats

### `FaceCacheStats`
```ts
type FaceCacheStats = {
  size: number        // number of face handles currently in cache
  hits: number        // cache hits since construction
  misses: number      // cache misses since construction
  evictions: number   // faces evicted (LRU) or re-registered since construction
}
```

### `getFaceCacheStats(): FaceCacheStats`
Returns a snapshot of the global face cache stats (mirrors the most recently synced `WasmFontEngine` instance's stats).

---

## WasmFontEngine

### `FaceCacheOptions`
```ts
type FaceCacheOptions = {
  maxEntries?: number   // default 20; set to ≤0 to disable caching (create-shape-drop per call)
}
```

### `FallbackShapeInput`
```ts
type FallbackShapeInput = {
  fontId: string
  text: string
  font: Font
  fontBytes: Uint8Array
}
```
Passed to the user-supplied `fallbackShaper` when the WASM face API is not available.

### `WasmFontEngineOptions`
```ts
type WasmFontEngineOptions = {
  faceCache?: FaceCacheOptions
  fallbackShaper?: (input: FallbackShapeInput) => Glyph[]
}
```

### `WasmFontEngine`
Implements `FontEngine` from `@paragraf/font-engine`.

```ts
class WasmFontEngine {
  constructor(wasm: unknown, options?: WasmFontEngineOptions)

  // Load font file from disk (Node.js only — uses fs.readFileSync).
  loadFont(id: string, path: string): Promise<void>

  // Register raw font bytes. Browser-safe alternative to loadFont.
  loadFontBytes(id: string, bytes: Uint8Array): void

  // Shape `text` using rustybuzz. Returns per-glyph info in font units.
  // Uses face-API path → fallback shaper path → legacy shape_text_wasm, in that order.
  glyphsForString(fontId: string, text: string, font?: Font): Glyph[]

  // Identity — GSUB ligatures applied by rustybuzz at shape time.
  applyLigatures(fontId: string, glyphs: Glyph[]): Glyph[]

  // Identity — GSUB sups/subs applied by rustybuzz at shape time via Font.variant.
  applySingleSubstitution(fontId: string, glyphs: Glyph[], featureTag: 'sups' | 'subs'): Glyph[]

  // Always returns 0 — GPOS kern included in advance width from rustybuzz.
  getKerning(fontId: string, glyph1: Glyph, glyph2: Glyph): number

  // Get glyph outline at (x, y) scaled to fontSize. Returns GlyphPath with toSVG().
  getGlyphPath(fontId: string, glyph: Glyph, x: number, y: number, fontSize: number): GlyphPath

  // Get font-level metrics for fontId at fontSize.
  getFontMetrics(fontId: string, fontSize: number, variant?: 'normal' | 'superscript' | 'subscript'): FontMetrics

  // Return current face cache stats for this instance.
  getFaceCacheStats(): FaceCacheStats

  // Drop all cached face handles and clear font bytes. Call before discarding instance.
  dispose(): void
}
```

---

## Binary WASM Bridge

### `serializeNodesToBinary(nodes: Node[]): [Float64Array, Uint8Array]`
Serialise a `Node[]` to compact binary format for `tracebackWasmBinary`.

Layout:
- `Float64Array`: 4 f64 values per node `[width, param1, param2, param3]`
  - Box: `[width, 0, 0, 0]`
  - Glue: `[width, stretch, shrink, 0]` (±Infinity → ±1e30)
  - Penalty: `[width, penalty, 0, 0]` (±Infinity → ±1e30)
- `Uint8Array`: type + flags per node (bits 0–3 = type, bits 4–7 = kind/flagged)

### `tracebackWasmBinary(wasm, nodes, lineWidth, tolerance, emergencyStretch?, looseness?, widowPenalty?, orphanPenalty?, consecutiveHyphenLimit?, lineWidths?, runtPenalty?, singleLinePenalty?): any`
Call the WASM `traceback_wasm_binary` entry point with binary-serialised nodes.

Parameters:
- `wasm`: wasm-bindgen module (from `loadShapingWasm()`)
- `nodes: Node[]`
- `lineWidth: number` — uniform line width in points
- `tolerance: number`
- `emergencyStretch?: number` — default 0
- `looseness?: number` — default 0
- `widowPenalty?: number` — **@deprecated** Use `runtPenalty` instead. Default 0
- `orphanPenalty?: number` — **@deprecated** Use `singleLinePenalty` instead. Default 0
- `consecutiveHyphenLimit?: number` — default 0
- `lineWidths?: number[]` — per-line widths override; default `[]`
- `runtPenalty?: number` — canonical name for widowPenalty. Takes precedence when both provided.
- `singleLinePenalty?: number` — canonical name for orphanPenalty. Takes precedence when both provided.

Returns: `{ ok: { breaks: LineBreak[], usedEmergency: boolean } }` or `{ error: string }`.

Last break's `ratio` is clamped to `0` (forced-break line; Rust returns ≈1e-28 due to ∞ sentinel).
