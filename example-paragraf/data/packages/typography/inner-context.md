# typography — Inner Context

---
type: inner-context
package: @paragraf/typography
layer: L3
updated: 260416-1955
---

## Role

Top-level paragraph and document compositor. Owns the full pipeline from raw `ParagraphInput` → `ComposedParagraph` → `RenderedDocument`. The highest-level package before L4 templates:

- **Paragraph composition**: Knuth-Plass line-breaking with WASM acceleration and TS fallback.
- **Optical Margin Alignment (OMA)**: Two-pass recompose with per-line protrusion table.
- **Document model**: Multi-paragraph, multi-frame, multi-page composition (`composeDocument`) and layout (`layoutDocument`).
- **Measurer**: Word measurement with an LRU word-measurement cache.
- **BiDi**: Paragraph-level direction detection (WASM full UBA or TS P2/P3 fallback).

---

## Process Rules

- `createParagraphComposer(registry, options?)` is the primary factory. Returns a `ParagraphComposer` whose `compose()` is synchronous. `await` is only needed for initial setup (language loading).
- WASM auto-detected at module init: `_wasm = loadShapingWasm()`. Failures silently set `_wasm = null` → TS fallback. Force TS: `options.useWasm = false`.
- `wasmStatus()` returns `'loaded' | 'absent' | 'error'`. Use to distinguish clean fallback from broken build.
- Measurement cache is module-level (`_measureCacheStore`, `_measureCacheStats`). Shared across all composer instances in the same process. `clearMeasureCache()` resets it.
- Cache key includes: `[word, fontId, size, weight, style, stretch, letterSpacing, variant, featureSetId]`. RTL paragraphs emit a one-time console.warn (script/direction not in key yet).
- `featureSetId` can be static (string) or dynamic (resolver function); controls GSUB feature grouping.
- OMA is two-pass: pass 1 breaks the paragraph at nominal `lineWidth`; pass 2 rebreaks with per-line adjusted widths from `buildOmaAdjustments`. Only `xOffset` and `rightProtrusion` are stamped onto the pass-2 lines — not the pass-1 lines.
- `lineHeight` override on `ParagraphInput` stamps a fixed leading onto every `ComposedLine`, bypassing font-metric-derived values.
- Document layout flows lines top-to-bottom, column-by-column, frame-by-frame. A line too tall for the remaining space is force-placed (guarantees termination).
- `composeDocument` warns (console.warn) when frames have different column widths. Use `deriveLineWidths()` to pre-assign per-paragraph widths.
- RTL paragraphs bypass hyphenation; spans are not supported in RTL (throws).
- BiDi determination: WASM calls `analyze_bidi`; TS fallback scans for first strong directional character (P2/P3 of UBA).

---

## Package Constraints

- **Layer L3**: may import from L0–L2. Must **never** import from L3+ packages (render-pdf, template, compile). Runtime deps include all five lower layers.
- `@paragraf/render-core` is a runtime dep (for `layoutDocument`). The document types (`BaselineGrid`, `Frame`, `RenderedItem`, etc.) are re-exported from typography so consumers need not add render-core directly.
- `_wasm`, `_wasmError`, `_rtlFallbackWarnIssued`, `_nonLtrCacheWarnIssued`, `_measureCacheStore`, `_measureCacheStats`, `_measureCacheConfig` are module-level singletons. They persist across test runs within a single vitest worker. Call `clearMeasureCache()` in test teardown (resets stats to zero and clears the store).
- `looseness` is clamped to integer before passing to WASM (`Math.trunc`).
- RTL paragraphs with `spans` input throw. Only plain `text` supported for RTL.
- **featureSetId is resolved in precedence order**: `featureSetIdResolver` → `featureConfig` (via `featureSetIdFromConfig`) → `featureSetId` (string) → `'__default-feature-set__'`. Use `featureSetIdFromConfig(config)` to derive a deterministic, registry-free ID from a `FeatureConfig` object. No registration step required — same config always produces the same string across callers and process restarts. (D001 locked 260430)

---

## Consistency Checks

- `ParagraphComposer.measurer` must be the same `Measurer` instance used internally — callers pass it to `layoutDocument` to avoid a second font-cache lookup.
- `composeDocument` overrides `lineWidth` with `colWidth(frames[0])` after merging `styleDefaults`. Per-paragraph `lineWidth` is therefore lost unless `deriveLineWidths()` is called first.
- All re-exported document types (`BaselineGrid`, `Frame`, `RenderedItem`, `RenderedPage`, `RenderedDocument`) must match those in `@paragraf/render-core`.
