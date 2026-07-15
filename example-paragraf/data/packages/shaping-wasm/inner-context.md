# shaping-wasm — Inner Context

---
type: inner-context
package: @paragraf/shaping-wasm
layer: L2
updated: 260416-1945
---

## Role

Rust/WASM font shaping engine. Bridges the JS typesetting pipeline to a `rustybuzz`-backed Rust/WASM module for high-quality OpenType shaping. Also provides binary serialisation helpers to push `Node[]` across the WASM boundary with minimal JSON overhead.

Implements the `FontEngine` interface from `@paragraf/font-engine` via `WasmFontEngine`.

---

## Process Rules

- `WasmFontEngine` is the only implementation of `FontEngine` that ships in a published package.
- Font bytes are held JS-side; WASM holds parsed face handles (resource handles). JS owns both lifetimes.
- Face handle cache is LRU, bounded by `maxEntries`. When `maxEntries <= 0` the cache is disabled: create-shape-drop in one call path.
- `loadFont(id, path)` uses `fs.readFileSync` — Node.js only. Browser callers must use `loadFontBytes(id, bytes)` with bytes from `fetch`.
- `glyphsForString` has three shaping paths:
  1. Face-API path (`hasFaceApi = true`): uses `create_face` / `shape_with_face` / `drop_face`.
  2. Fallback shaper path: delegates to user-supplied `fallbackShaper` function.
  3. Legacy path: calls `shape_text_wasm` (single-call, stateless).
- `applyLigatures`, `applySingleSubstitution`, `getKerning` are all identity/no-ops: rustybuzz applies GSUB/GPOS at shape time.
- `loadShapingWasm()` uses `createRequire` (CJS require) to load the wasm-pack output. Must be called before constructing `WasmFontEngine` with the face API.
- `tracebackWasmBinary` clamps the last break's ratio to 0 (the last line is always a forced break; Rust returns ≈1e-28 due to ∞ sentinel).

---

## Package Constraints

- **Layer L2**: may import from L0 (`@paragraf/types`) and L1 (`@paragraf/font-engine`). Never from L2+.
- WASM binary (`wasm/pkg/`) is built by `wasm-pack`; not TypeScript-compiled. Must be present before tests run.
- `loadFont` is Node.js-only; `loadFontBytes` is the browser-safe path.
- `±Infinity` penalty/stretch values are mapped to `±1e30` before passing to WASM to avoid NaN from `∞ − ∞` in prefix-sum subtraction.

---

## Consistency Checks

- Face handle count (`faceHandlesByFontId.size`) must equal cache `size` stat.
- Every `create_face` must have a corresponding `drop_face` — either in `evictIfNeeded`, `dispose`, or the one-shot path's `finally` block.
- `globalFaceCacheStats` is updated via `syncGlobalStats()` after every mutation.
