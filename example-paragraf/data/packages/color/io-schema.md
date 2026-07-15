# color — I/O Schema

---
type: io-schema
package: @paragraf/color
updated: 260503-1630
---

## Purpose

All public types, interfaces, and functions exported by `@paragraf/color`.

---

## Color Space Types

### `RGBColor`
```ts
interface RGBColor {
  readonly r: number   // [0, 1]
  readonly g: number   // [0, 1]
  readonly b: number   // [0, 1]
}
```

### `CMYKColor`
```ts
interface CMYKColor {
  readonly c: number   // [0, 1]
  readonly m: number   // [0, 1]
  readonly y: number   // [0, 1]
  readonly k: number   // [0, 1]
}
```

### `LabColor`
```ts
interface LabColor {
  readonly L: number   // [0, 100]
  readonly a: number   // typically [-128, 127]
  readonly b: number   // typically [-128, 127]
}
```

### `GrayColor`
```ts
interface GrayColor {
  readonly gray: number   // [0, 1]
}
```

### `RenderingIntent`
```ts
type RenderingIntent = 'perceptual' | 'relative' | 'saturation' | 'absolute'
```

### `TaggedColor`
```ts
interface TaggedColor {
  source: RGBColor                              // original device-RGB value (pre-transform)
  transformed: CMYKColor | RGBColor | GrayColor | null   // post-ICC-transform result; null if no transform active
  profileBytes: Uint8Array                      // raw ICC profile bytes for PDF/X OutputIntent embedding
}
```

---

## ICC Profile Types

### `ColorSpace`
```ts
type ColorSpace = 'RGB' | 'CMYK' | 'Lab' | 'Gray'
```

### `PcsSpace`
```ts
type PcsSpace = 'XYZ' | 'Lab'
```

### `TrcCurve`
```ts
type TrcCurve =
  | { kind: 'gamma'; gamma: number }
  | { kind: 'lut'; values: Float64Array }
  | { kind: 'linear' }
```
Tone reproduction curve — represents per-channel linearization.

### `XYZValue`
```ts
interface XYZValue {
  x: number
  y: number
  z: number
}
```

### `Mft2Tag`
```ts
interface Mft2Tag {
  inChannels: number
  outChannels: number
  gridPoints: number
  matrix: number[]              // 9-element row-major 3×3 (only meaningful when inChannels === 3)
  inputCurves: Float64Array[]   // per-input-channel 1D curves, normalized to [0, 1]
  clut: Float64Array            // flattened CLUT: indexed by [gridIdx * outChannels + outCh]
  outputCurves: Float64Array[]  // per-output-channel 1D curves, normalized to [0, 1]
}
```
Parsed mft2 (16-bit LUT) tag — used for A2B0 / B2A0 CLUT transforms in ICC profiles.

### `ColorProfile`
```ts
interface ColorProfile {
  readonly name: string
  readonly colorSpace: ColorSpace
  readonly pcs: PcsSpace
  readonly renderingIntent: number
  readonly whitePoint: XYZValue
  readonly matrix?: { r: XYZValue; g: XYZValue; b: XYZValue }   // present on RGB profiles
  readonly trc?: [TrcCurve, TrcCurve, TrcCurve]                  // per-channel TRC; present on RGB matrix profiles
  readonly a2b0?: Mft2Tag                                         // device → PCS LUT
  readonly b2a0?: Mft2Tag                                         // PCS → device LUT
  readonly bytes: Uint8Array                                      // raw ICC profile bytes for PDF embedding
}
```
Fully parsed ICC v2/v4 profile.

---

## Profile Functions

### `parseIccProfile(bytes: Uint8Array): ColorProfile`
Parse raw ICC profile bytes into a `ColorProfile`. Supports RGB matrix + TRC and CLUT-based profiles.
ICC v4 `para` tags (parametric curves, types 0–4) are sampled into a 1024-point LUT using `sampleParametricCurve`.

### `loadProfile(path: string): Promise<ColorProfile>`
Load an ICC profile from disk and parse it. **Node.js only** — uses `node:fs/promises`.

### `buildSrgbProfileBytes(): Uint8Array`
Synthesize a minimal sRGB ICC profile in memory. No disk I/O.

### `loadBuiltinSrgb(): ColorProfile`
Return a `ColorProfile` for the built-in synthesized sRGB profile.

### `sampleParametricCurve(fnType: number, params: number[], x: number): number`
Evaluate an ICC v4 parametric curve (para tag §10.15) at `x ∈ [0, 1]`.
Exported for testing and advanced use.

| fnType | Formula                                           | params              |
|--------|---------------------------------------------------|---------------------|
| 0      | `Y = X^g`                                         | `[g]`               |
| 1      | `Y = (aX+b)^g` if `X ≥ −b/a`, else `0`           | `[g, a, b]`         |
| 2      | `Y = (aX+b)^g + c` if `X ≥ −b/a`, else `c`       | `[g, a, b, c]`      |
| 3      | `Y = (aX+b)^g` if `X ≥ d`, else `cX`             | `[g, a, b, c, d]`   |
| 4      | `Y = (aX+b)^g + e` if `X ≥ d`, else `cX + f`     | `[g, a, b, c, d, e, f]` |

---

## Transform Types and Functions

### `ColorTransform`
```ts
interface ColorTransform {
  apply(input: number[]): number[]   // normalized channel arrays in [0, 1]
}
```
Compiled color transform between two ICC profiles.

### `createTransform(source: ColorProfile, dest: ColorProfile, intent?: RenderingIntent): ColorTransform`
Compile a `ColorTransform` from `source` to `dest` profile using the given rendering intent (default: `'perceptual'`).

### `applyTrcForward(trc: TrcCurve, v: number): number`
Apply a TRC curve in the forward (device → linear) direction. Exported for testing and advanced use.

### `xyzToIccLab(xyz: number[], wp: XYZValue): number[]`
Convert CIEXYZ (D50-adapted) to ICC-normalized Lab `[L/100, (a+128)/255, (b+128)/255]`. Exported for testing and advanced use.

---

## LUT Evaluation Functions

Exported for testing and advanced use. Internal to the transform pipeline.

### `eval1DCurve(values: Float64Array, t: number): number`
Interpolate a 1D LUT at position `t` in `[0, 1]`.

### `evalClutTetrahedral(clut: Float64Array, inCh: number, outCh: number, gridPoints: number, input: number[]): number[]`
Evaluate a CLUT using tetrahedral interpolation. `inCh` must be 3 for the tetrahedral path; falls back to trilinear for other channel counts.

### `evalLutMft2(tag: Mft2Tag, input: number[]): number[]`
Evaluate a full mft2 LUT tag (input curves → CLUT → output curves).

---

## Color Manager

### `OutputIntent`
```ts
interface OutputIntent {
  profile: ColorProfile
  condition: string   // e.g. 'FOGRA39', 'CGATS TR 001'
}
```

### `ColorManager`
```ts
interface ColorManager {
  loadProfile(path: string): Promise<ColorProfile>          // load + cache profile from disk (Node.js only)
  loadBuiltinSrgb(): ColorProfile                           // return + cache built-in sRGB profile
  createTransform(source: ColorProfile, dest: ColorProfile, intent?: RenderingIntent): ColorTransform
  getOutputIntent(profile: ColorProfile, condition: string): OutputIntent
}
```
Profile-caching color management entry point. Cache is per-instance, keyed by file path.

### `createColorManager(): ColorManager`
Create a new `ColorManager` instance with an empty profile cache.
