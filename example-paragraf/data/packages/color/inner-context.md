# color — Inner Context

---
type: inner-context
package: @paragraf/color
updated: 260416-1930
---

## Purpose

Package-level consistency and accuracy checker. Consulted at start and end of every inner-loop task for this package.

## Role

ICC color management for the paragraf pipeline. Parses ICC v2/v4 profiles, applies color transforms (RGB → CMYK, RGB → Lab, etc.), and provides a `ColorManager` for profile caching and `OutputIntent` generation for PDF/X embedding.

Responsible for:
- Color space types (`RGBColor`, `CMYKColor`, `LabColor`, `GrayColor`, `TaggedColor`)
- Rendering intent type (`RenderingIntent`)
- ICC profile parsing (`ColorProfile`, `ColorSpace`, `PcsSpace`, `TrcCurve`, `XYZValue`, `Mft2Tag`)
- Profile loading from disk (`loadProfile`) and built-in sRGB (`loadBuiltinSrgb`, `buildSrgbProfileBytes`)
- Color transforms (`ColorTransform`, `createTransform`)
- TRC and LUT evaluation (`applyTrcForward`, `xyzToIccLab`, `eval1DCurve`, `evalClutTetrahedral`, `evalLutMft2`)
- Color manager (`ColorManager`, `OutputIntent`, `createColorManager`)

Does **not** own:
- PDF embedding mechanics (owned by `@paragraf/render-pdf`)
- Rendering pipeline integration (owned by `@paragraf/render-core`, `@paragraf/render-pdf`)
- Font color or text color styling (owned by `@paragraf/style`)

## Process Rules

- TDD is mandatory — tests before tasks, no exceptions unless explicitly overridden for a specific workId
- Division of labour: test descriptions are human-authored; test code and task implementations are LLM-generated
- Zero `@paragraf/*` runtime dependencies — this package must not import another paragraf package
- Layer L0: no layer import rule applies (base layer, alongside `@paragraf/types`)
- `loadProfile` uses `node:fs/promises` — package is not browser-safe as shipped; browser callers must supply profile bytes directly
- LUT evaluation functions (`eval1DCurve`, `evalClutTetrahedral`, `evalLutMft2`) are public for testing and advanced use but are internal to the transform pipeline

---

## Consistency Checks (Copilot-executed)

Pre-task:
- [ ] Active plan workIds exist in `docs/work-pool.md`
- [ ] io-schema.md types match exported TypeScript types in `src/index.ts`
- [ ] No runtime imports of `@paragraf/*` packages

Post-task:
- [ ] io-schema.md updated if public types changed
- [ ] io-schema.md changes flagged for sync to project-level `docs/io-schemas.md`
