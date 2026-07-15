# render-pdf — Inner Context

---
type: inner-context
package: @paragraf/render-pdf
layer: L3
updated: 260418-1700
---

## Role

PDF rendering backend. Converts `RenderedParagraph` or `RenderedDocument` to a `Buffer` containing a valid PDF binary. Glyphs are drawn as vector paths (not embedded font text), preserving all GSUB substitutions. An optional invisible text overlay enables copy-paste / accessibility without affecting visual output.

---

## Process Rules

- `pdfkit` is loaded lazily via `createRequire` (CJS shim) on first call. Avoids adding it to the ESM import graph.
- `upmCache` is a module-level `Map<fontId, unitsPerEm>`. `clearPdfCaches()` resets it. Call in tests to avoid cross-test contamination.
- Glyph drawing follows the same path-command loop as `renderToSvg` / `renderToCanvas` (from render-core): `moveTo` / `lineTo` / `bezierCurveTo` / `quadraticCurveTo` / `closePath` / `fill`.
- `doc.save()` / `doc.restore()` wrap each glyph path to isolate fill state.
- Selectable mode (`selectable: true`): after drawing vector paths for a segment, `emitInvisibleSegment` writes a `BT…ET` block with `Tr=3` (invisible fill+stroke) and encodes glyphs via pdfkit's `font.encode()`. The ToUnicode CMap maps hex glyph IDs back to Unicode, enabling copy-paste.
- `renderToPdf` creates a single PDFDocument; `renderDocumentToPdf` uses `autoFirstPage: false` and calls `doc.addPage()` for each page in the `RenderedDocument`.
- `selectable: true` requires `fontRegistry`. Both functions reject with an error if fontRegistry is missing.
- A4 defaults: `width=595.28`, `height=841.89` (points). Callers supply explicit dimensions for other page sizes.
- Letter spacing is applied between glyphs but not after the last glyph.

---

## Package Constraints

- Layer L3: may import from L0–L2. Runtime deps: `@paragraf/types`, `@paragraf/color`, `@paragraf/font-engine`, `@paragraf/render-core`, `pdfkit`.
- Never import `@paragraf/typography` or `@paragraf/shaping-wasm` at runtime (those are L3/L2; there is no upward dep).
- `@paragraf/render-core` pinned to `>=0.3.1 <0.4.0` (exact feature contract for `getAndSubstituteGlyphs`).
- pdfkit is a third-party CJS package loaded via `createRequire`. Do not use a static ESM import.
- **Output is vector-path (not TJ text operators). PDF/X conformance is deferred to v1.0.** `selectable: true` adds a separate invisible BT…ET overlay for copy-paste / accessibility, but does not make the PDF PDF/X-conformant. Do not add TJ text emission without first wiring veraPDF CI validation.
- Invisible text CTM trick: must use `Tm 1 0 0 -1 x y` (not `Td`) to counter-flip pdfkit's y-flip CTM.
- `emitOutputIntent`: uses `doc._root.data.OutputIntents = [intentRef]` — internal pdfkit API. Must be called after all page content and before `doc.end()`. Creates two PDF objects: an ICC stream ref (via `doc.ref()` + `.write()` + `.end()`) and an OutputIntent dict ref (via `doc.ref()` + `.end()` only). Exactly one `emitOutputIntent` call per document.

---

## Consistency Checks

- `emitInvisibleSegment` must be called AFTER glyph paths for a segment are drawn (clean graphics state).
- Font must be registered in the page resource dict before `addContent` emits the BT block.
- `applyMetadata` must be called before `doc.end()`.
