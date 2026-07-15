# Input / Output Schemas

---
type: io-schemas
version: 3
updated: 260428-1000
---

## Purpose

Navigation index. Full field-by-field definitions live in each package's `io-schema.md`. Read this file to find the right per-package file; read that file for the actual types and signatures.

---

## Package index

| Layer | Package | npm name | What it exports | Schema file |
|---|---|---|---|---|
| L0 | types | `@paragraf/types` | All shared primitives, node types, font types, function types (MeasureText, GlueSpaceFn, etc.) | `data/packages/types/io-schema.md` |
| L0 | color | `@paragraf/color` | ICC color types, ColorProfile, ColorTransform, ColorManager, OutputIntent, parsing/loading functions | `data/packages/color/io-schema.md` |
| L1 | linebreak | `@paragraf/linebreak` | Knuth-Plass API: computeBreakpoints, traceback, buildNodeSequence, composeParagraph, hyphenation helpers | `data/packages/linebreak/io-schema.md` |
| L1 | font-engine | `@paragraf/font-engine` | FontEngine interface, FontkitEngine, createMeasurer, createFontRegistry, glyph types, font loading helpers | `data/packages/font-engine/io-schema.md` |
| L1 | style | `@paragraf/style` | ParagraphStyleDef, CharStyleDef, resolved style types, StyleRegistry, defineStyles | `data/packages/style/io-schema.md` |
| L1 | layout | `@paragraf/layout` | PageLayout, page sizes, Dimension, unit converters (px, parseDimension), Frame helpers | `data/packages/layout/io-schema.md` |
| L2 | shaping-wasm | `@paragraf/shaping-wasm` | WasmFontEngine class, binary bridge (serializeNodesToBinary, tracebackWasmBinary), loadShapingWasm | `data/packages/shaping-wasm/io-schema.md` |
| L2 | color-wasm | `@paragraf/color-wasm` | WasmColorTransform, createWasmTransform, loadColorWasm — WASM-accelerated ICC color transforms | `data/packages/color-wasm/io-schema.md` |
| L2 | render-core | `@paragraf/render-core` | RenderedParagraph/Document types, layoutParagraph, getAndSubstituteGlyphs, renderToSvg, renderToCanvas | `data/packages/render-core/io-schema.md` |
| L3 | typography | `@paragraf/typography` | ParagraphComposer, composeDocument, layoutDocument, OMA helpers, measure cache controls | `data/packages/typography/io-schema.md` |
| L3 | render-pdf | `@paragraf/render-pdf` | renderToPdf, renderDocumentToPdf, PdfOptions, DocumentPdfOptions | `data/packages/render-pdf/io-schema.md` |
| L4 | template | `@paragraf/template` | Template type, defineTemplate, ContentSlot, TemplateFonts, parseTokens | `data/packages/template/io-schema.md` |
| L4 | compile | `@paragraf/compile` | compile, compileBatch, CompileOptions, CompileResult, font registry helpers + re-exported lower-layer APIs | `data/packages/compile/io-schema.md` |
