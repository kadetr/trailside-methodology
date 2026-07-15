---
type: data
version: 3
updated: 260503-1730
actions: [read, append, update]
frequency: [on-demand, per-session]
---

# props — architecture (outer loop)

| Field        | Values                                      | Purpose                                               |
| ------------ | ------------------------------------------- | ----------------------------------------------------- |
| `package`    | string                                      | short package name                                    |
| `layer`      | 0–4 \| App                                  | architectural layer number                            |
| `dependency` | list                                        | key @paragraf/* architectural dependencies (not exhaustive) |
| `status`     | `stable` \| `experimental` \| `deprecated`  | package maturity                                      |
| `third-party-deps` | list                                  | third-party dependencies                              |

# props — io-schemas (inner loop / on-demand)

| Field         | Values       | Purpose                                  |
| ------------- | ------------ | ---------------------------------------- |
| `layer`       | L0–L4 \| App | architectural layer                      |
| `npm-name`    | string       | full npm package name                    |
| `exports`     | string       | summary of what the package exports      |
| `schema-file` | path         | link to per-package io-schema.md         |

# rules

- Architecture section used at outer loop / session boundary — layer and structural deps
- Dependency section used during inner loops — full runtime deps per package
- IO-schemas section is a navigation index only — full definitions live in per-package io-schema.md
- Dev dependencies not listed here — see root package.json devDependencies
- Optional deps noted with symbol in table (e.g. ‡ for graceful fallback)
- Updated when a package is added, removed, its layer changes, its imports change, or its exported types change

---

# architecture & dependency — package layer map

| package | layer | dependency | status | third-party-deps |
|---|---|---|---|---|
| @paragraf/types | L0 | — | stable | — |
| @paragraf/color | L0 | - | stable | — |
| @paragraf/style | L1 | @paragraf/types | stable | — |
| @paragraf/linebreak | L1 | @paragraf/types | stable | hyphen, fontkit |
| @paragraf/font-engine | L1 | @paragraf/types | stable | fontkit |
| @paragraf/layout | L1 | @paragraf/types | stable | — |
| @paragraf/render-core | L2 | @paragraf/types, @paragraf/font-engine | stable | - |
| @paragraf/shaping-wasm | L2 | @paragraf/types, @paragraf/font-engine | experimental | Rust-WASM |
| @paragraf/color-wasm | L2 | @paragraf/color | experimental | Rust-WASM |
| @paragraf/typography | L3 | @paragraf/types, @paragraf/linebreak, @paragraf/font-engine, @paragraf/render-core, @paragraf/shaping-wasm | stable | - |
| @paragraf/render-pdf | L3 | @paragraf/types, @paragraf/render-core, @paragraf/font-engine | stable | pdfkit |
| @paragraf/template | L4 | @paragraf/types, @paragraf/layout, @paragraf/style | stable | - |
| @paragraf/compile | L4 | @paragraf/types, @paragraf/color, @paragraf/linebreak, @paragraf/layout, @paragraf/style, @paragraf/font-engine, @paragraf/render-core, @paragraf/shaping-wasm, @paragraf/color-wasm, @paragraf/render-pdf, @paragraf/typography, @paragraf/template| stable | - |

‡ optional / graceful-fallback dependency

---

# io-schemas — navigation index

| layer | npm-name | exports | schema-file |
|---|---|---|---|
| L0 | @paragraf/types | core type primitives, shared across all layers | data/packages/types/io-schema.md |
| L0 | @paragraf/color | RGBColor, CMYKColor, TaggedColor, RenderingIntent, OutputIntent, ColorTransform, ICC profile parsing | data/packages/color/io-schema.md |
| L0 | @paragraf/style | StyleBlock, FontFeatures, featureSetId | data/packages/style/io-schema.md |
| L0 | @paragraf/linebreak | Knuth-Plass line-breaking primitives | data/packages/linebreak/io-schema.md |
| L1 | @paragraf/font-engine | Measurer, FontFace, font registry types | data/packages/font-engine/io-schema.md |
| L1 | @paragraf/layout | Frame, Page, layout primitives | data/packages/layout/io-schema.md |
| L1 | @paragraf/template | TemplateLayout, template composition | data/packages/template/io-schema.md |
| L2 | @paragraf/shaping-wasm | loadShapingWasm, ShapingResult (WASM-backed) | data/packages/shaping-wasm/io-schema.md |
| L2 | @paragraf/color-wasm | loadColorWasm, WasmColorTransform, createWasmTransform | data/packages/color-wasm/io-schema.md |
| L3 | @paragraf/typography | ParagraphComposer, MeasureCacheOptions | data/packages/typography/io-schema.md |
| L3 | @paragraf/render-core | render primitives, draw context | data/packages/render-core/io-schema.md |
| L3 | @paragraf/render-pdf | renderToPdf, renderDocumentToPdf, PdfOptions, DocumentPdfOptions, emitOutputIntent | data/packages/render-pdf/io-schema.md |
| L4 | @paragraf/compile | compile, CompileOptions, compliance mode | data/packages/compile/io-schema.md |
| App | @paragraf/studio | (TBD — visual editor host) | data/packages/studio/io-schema.md |

---

# layer rules

- **L0 — foundational types and pure-data utilities.** No I/O, no native deps. May export types and pure functions.
- **L1 — primitives with shaped data.** Operates on L0 types. May have third-party deps for parsing/measurement.
- **L2 — accelerated backends.** WASM/Rust-backed implementations of L0/L1 interfaces. Optional, opt-in. Always shipped with a JS fallback.
- **L3 — composition and rendering.** Combines L0–L2 to produce output. May depend on optional L2 packages (graceful fallback).
- **L4 — orchestration.** Public API surface. Wraps L3 packages with high-level configuration (compliance modes, defaults).
- **App — application-tier.** Browser/desktop hosts. May depend on any layer.

A package may NOT import from a higher layer than its own. L2 may import from L0/L1 but not L3+. Optional deps noted with ‡ are runtime-resolved and may be absent.
