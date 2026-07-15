# compile — Inner Context

---
type: inner-context
package: @paragraf/compile
layer: L4
updated: 260418-1800
---

## Role

Top-level document compile pipeline. The user-facing entry point for the entire paragraf stack. Merges a validated `Template`, a data record, font files, layout geometry, and styles into a rendered PDF, SVG, or `RenderedDocument`.

Owns an 11-step pipeline:
1. Resolve fonts → `FontRegistry`
2. Build composer + FontEngine
3. Resolve layout → `PageLayout` → `Frame[]`
4. Resolve styles → `StyleRegistry`
5. Resolve data → apply `normalize()` if provided
6. Interpolate slots → per slot: resolve bindings, apply `onMissing`
7. Build `ParagraphInput[]` from resolved slots
8. Ensure languages (call `ensureLanguage` for non-default)
9. Compose → `composeDocument`
10. Layout → `layoutDocument`; count overflow
11. Render output → PDF `Buffer` / SVG `string` / `RenderedDocument`

Also owns the font variant convention table (`VARIANT_CONVENTIONS`, 18 keys), CSS-nearest-weight variant selection, and binding interpolation (`resolveText`).

Serves as the "batteries-included" package: re-exports the full public API of all lower layers so consumers need only one `@paragraf/compile` dependency.

---

## Process Rules

- `compile()` is async and resolves to `CompileResult`. All 11 steps run sequentially for a single record.
- `compileBatch()` processes records with bounded concurrency (default 4). Each record runs `compile()` independently. Results include per-record `result` or `error` fields.
- `maxPages` validation: throws `RangeError` if `< 1`. Default 100.
- `concurrency` validation: throws `RangeError` if `< 1`. Default 4.
- `selectable: true` with `output !== 'pdf'`: emits a console.warn and has no effect (not an error).
- `outputIntent` with `output !== 'pdf'`: emits a console.warn and has no effect. Same pattern as selectable guard.
- `shaping: 'auto'` auto-detects WASM; `'wasm'` forces it (silent fallback if not built); `'fontkit'` always uses TS path.
- `@paragraf/shaping-wasm` is an `optionalDependency`. Its absence is handled gracefully (WASM status `'absent'`).
- Font variant resolution: string shorthand keys are looked up in `VARIANT_CONVENTIONS` (18 keys). Unknown keys emit console.warn and default to weight 400, style 'normal'.
- `resolveText` is all-or-nothing: if any binding in a slot resolves to `null`/`undefined`, the entire slot result is `null` and `onMissing` applies.
- `buildFontRegistry`: resolves relative font paths against `basePath` (default `process.cwd()`). Throws if a file does not exist.
- Module-level `_warnedSpacing` and `_warnedHyphenation` Sets deduplicate development-time warnings in high-volume batch runs.

---

## Package Constraints

- Layer L4: the top of the stack. May import from any lower layer. No package may depend on `@paragraf/compile`.
- **Direct `@paragraf/linebreak` dependency is intentional**: compile is the batteries-included layer and re-exports the full linebreak public API (`computeBreakpoints`, `traceback`, `buildNodeSequence`, `composeParagraph`, hyphenation helpers) so consumers need only one `@paragraf/compile` dependency. This is not an accidental bypass of the L3 abstraction.
- `@paragraf/shaping-wasm` is optional. Never assume it is present.
- `output: 'svg'` renders each page independently via `renderToSvg`; pages are joined by `'\n'`. One `<svg>` per page.
- `output: 'rendered'` returns the `RenderedDocument` directly with no further rendering.
- `basePath` defaults to `process.cwd()`. In browser environments, callers must supply pre-resolved absolute paths.

---

## Consistency Checks

- `CompileResult.metadata.shapingEngine` must match the actual backend used (`'wasm'` or `'fontkit'`).
- `metadata.overflowLines` counts lines that did not fit within `maxPages`, not paragraphs.
- `CompileBatchResult` must set exactly one of `result` or `error` per record.
- Re-exported APIs must match their origin packages' public API exactly (no silent narrowing).
