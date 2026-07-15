# color-wasm — I/O Schema

---
type: io-schema
package: @paragraf/color-wasm
updated: 260418-1900
---

## Purpose

All public types, interfaces, classes, and functions exported by `@paragraf/color-wasm`.

Re-exported from `@paragraf/color` (available to callers via `import type { OutputIntent } from '@paragraf/color-wasm'`): `OutputIntent`.

Consumed but not re-exported from `@paragraf/color`: `ColorProfile`, `ColorTransform`, `RenderingIntent`, `TrcCurve`, `XYZValue`.

---

## Loader

### `loadColorWasm(): unknown`
Load the compiled Rust/WASM module synchronously via `createRequire`.
Returns the raw wasm-bindgen JS module. Throws if `wasm/pkg/color_wasm.js` is not present (i.e. `wasm-pack` has not been run via `npm run build:wasm`).

Must be called once before calling `createWasmTransform`.

---

## WasmColorTransform

### `WasmColorTransform`
Implements `ColorTransform` from `@paragraf/color`. Drop-in replacement for the object returned by `createTransform`.

```ts
class WasmColorTransform implements ColorTransform {
  apply(input: number[]): number[]
  // input: normalized channel values in [0, 1]; caller must clamp
  // output: normalized channel values in [0, 1] (XYZ for matrix-only path;
  //         CMYK or other for LUT path; delegated for unsupported profiles)
}
```

---

## Factory

### `createWasmTransform`
```ts
function createWasmTransform(
  wasm: unknown,
  source: ColorProfile,
  dest: ColorProfile,
  intent?: RenderingIntent,   // default: 'perceptual'
): WasmColorTransform
```

Compile a `WasmColorTransform` from `source` to `dest` profile.

**Accelerated paths:**
| Source | Destination | Path |
|---|---|---|
| RGB matrix + TRC | RGB matrix | matrix-TRC (WASM: per-channel gamma + matrix multiply) |
| RGB matrix + TRC | CMYK/LUT (B2A0) | matrix-TRC + XYZ→Lab + CLUT (all WASM) |

**Fallback:** unsupported combinations (e.g. CMYK source with A2B0) delegate to `createTransform` from `@paragraf/color`. Return type is still `WasmColorTransform`.
