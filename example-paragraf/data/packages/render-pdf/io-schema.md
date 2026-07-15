# render-pdf — I/O Schema

---
type: io-schema
package: @paragraf/render-pdf
updated: 260419-0900
---

## Purpose

All public types, interfaces, and functions exported by `@paragraf/render-pdf`.

Types consumed but not re-exported: `RenderedParagraph`, `RenderedDocument`, `getAndSubstituteGlyphs` (from `@paragraf/render-core`), `FontEngine` (from `@paragraf/font-engine`), `FontRegistry` (from `@paragraf/types`), `WasmColorTransform`, `createWasmTransform`, `loadColorWasm` (from `@paragraf/color-wasm`).

Types re-exported from upstream: `OutputIntent` (from `@paragraf/color`) — available to callers as `import type { OutputIntent } from '@paragraf/render-pdf'`.

---

## Single-Paragraph Renderer

### `PdfOptions`
```ts
interface PdfOptions {
  width?: number           // page width in points; default 595.28 (A4)
  height?: number          // page height in points; default 841.89 (A4)
  fill?: string            // glyph fill color; default 'black'
  selectable?: boolean     // add invisible text overlay; default false
  fontRegistry?: FontRegistry  // required when selectable is true
  title?: string           // PDF Info dict title
  lang?: string            // PDF Info dict language tag
  compress?: boolean       // pdfkit compression; defaults to pdfkit's own default
  outputIntent?: OutputIntent  // embed ICC OutputIntent in PDF catalog (PDF/A, PDF/X)
  colorTransform?: ColorTransform  // optional ICC color transform; converts fill from sRGB to output color space
}
```

### `renderToPdf(rendered: RenderedParagraph, fontEngine: FontEngine, options?: PdfOptions): Promise<Buffer>`
Render a single composed paragraph to a PDF `Buffer`.

- Glyphs are drawn as vector paths (not embedded font text). GSUB substitutions (ligatures, sups/subs) are preserved.
- If `selectable: true`: emits an invisible `BT…ET` text overlay per segment for copy-paste support. `fontRegistry` must be provided or the promise rejects.
- PDF coordinates: origin at top-left, y increases downward (pdfkit convention). All input coordinates are in points.

---

## Document Renderer

### `DocumentPdfOptions`
```ts
interface DocumentPdfOptions {
  pageWidth?: number       // default 595.28 (A4)
  pageHeight?: number      // default 841.89 (A4)
  fill?: string            // default 'black'
  selectable?: boolean     // default false
  fontRegistry?: FontRegistry  // required when selectable is true
  title?: string
  lang?: string
  compress?: boolean
  outputIntent?: OutputIntent  // embed ICC OutputIntent in PDF catalog (PDF/A, PDF/X)
  colorTransform?: ColorTransform  // optional ICC color transform; converts fill from sRGB to output color space
}
```

### `renderDocumentToPdf(renderedDoc: RenderedDocument, fontEngine: FontEngine, options?: DocumentPdfOptions): Promise<Buffer>`
Render a multi-page `RenderedDocument` to a PDF `Buffer`.

- Uses `autoFirstPage: false`; calls `doc.addPage()` for each page in `renderedDoc.pages`.
- Each page's `items` (list of `RenderedItem`) are drawn in order.
- Same selectable behaviour as `renderToPdf`.

---

## Cache Utilities

### `clearPdfCaches(): void`
Clear the module-level `upmCache` (`Map<fontId, unitsPerEm>`). Call in test setup/teardown to avoid cross-test contamination.

---

## Internal Helpers (consumed, not exported)

- `parseCssToSrgb(css: string): [number, number, number] | null` — parses CSS color string (named, #hex, rgb()) to normalized sRGB [0,1] triplet.
- `applyFillTransform(transform: ColorTransform, fill: string): string | number[]` — converts a CSS fill string to a PDFKit-compatible CMYK array via the color transform; returns the original string on parse failure (safe passthrough).
