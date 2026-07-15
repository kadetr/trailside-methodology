# color-wasm — Inner Context

---
type: inner-context
package: @paragraf/color-wasm
layer: L2
updated: 260418-1900
---

## Role

Rust/WASM-accelerated ICC color transform backend. Bridges the JS color pipeline to a WASM module for high-performance matrix-TRC and tetrahedral-CLUT color transforms. Implements the `ColorTransform` interface from `@paragraf/color` via `WasmColorTransform`.

For unsupported profile combinations (e.g. CMYK source with A2B0 LUT), `WasmColorTransform` automatically delegates to the pure-TS `createTransform` from `@paragraf/color`.

---

## Process Rules

- `WasmColorTransform` is a drop-in replacement for the `ColorTransform` returned by `createTransform` in `@paragraf/color`. The interface is identical.
- `loadColorWasm()` uses `createRequire` (CJS require) to load the wasm-pack output. Must be called before constructing any `WasmColorTransform` via `createWasmTransform`.
- TRC linearization happens in TS (calling individual WASM functions per channel) before the matrix multiply. This avoids excess WASM boundary crossings for complex TRC dispatch.
- The `apply_matrix_gamma_trc` Rust export is called with gamma=1.0 when TRC has already been applied — it acts as a pure matrix multiply in that case.
- Input clamping to `[0, 1]` is the TS caller's responsibility. Rust does not re-validate.

---

## Package Constraints

- **Layer L2**: may import from L0 (`@paragraf/color`, `@paragraf/types`). Never from L1+.
- **`@paragraf/color` is NOT modified**: this package is purely additive. No existing packages change.
- **render-pdf and compile are unaffected**: neither package calls `createTransform` — they only embed raw ICC bytes.
- **WASM binary (`wasm/pkg/`) is built by `wasm-pack`**; not TypeScript-compiled. Must be present before tests run. Build: `npm run build:wasm`.
- **`Box<[f64]>` return type**: Rust functions returning `Box<[f64]>` produce `Float64Array` in JS via wasm-bindgen. Callers receive typed arrays.

---

## Rust WASM Exports

| Function | Purpose |
|---|---|
| `hello(name)` | Toolchain smoke test |
| `apply_gamma_trc(gamma, v)` | Power-law TRC: v^gamma |
| `eval_trc_lut(lut, t)` | 1D LUT evaluation (uniform grid, linear interp) |
| `apply_matrix_gamma_trc(gr, gg, gb, mat, r, g, b)` | Per-channel gamma + 3×3 matrix multiply → XYZ |
| `xyz_to_icc_lab(x, y, z, wp_x, wp_y, wp_z)` | XYZ → ICC-normalized Lab |
| `eval_clut_tetrahedral(clut, grid_points, out_ch, r, g, b)` | 3-input tetrahedral CLUT interpolation |

---

## Consistency Checks (Copilot-executed)

Pre-task:
- [ ] Active plan workIds exist in `docs/work-pool.md`
- [ ] io-schema.md types match exported TypeScript types in `src/index.ts`
- [ ] No L1+ imports
