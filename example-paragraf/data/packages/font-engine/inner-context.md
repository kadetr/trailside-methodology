# font-engine — Inner Context

---
type: inner-context
package: @paragraf/font-engine
updated: 260416-1800
---

## Purpose

Package-level consistency and accuracy checker. Consulted at start and end of every inner-loop task for this package.

## Role

Owns the JavaScript-side text shaping pipeline. Responsible for:
- Font loading and registration (by id)
- Glyph access and metrics via pluggable `FontEngine` backends
- Word measurement (`MeasureText`, `GlueSpaceFn`, `GetFontMetrics`) via `createMeasurer`
- Testing utilities (`mockMeasure`, `mockSpace`, `mockMetrics`)

Does **not** own:
- Word-level measurement cache policy (owned by `@paragraf/typography`)
- Benchmark ownership for cache effectiveness (owned by `@paragraf/typography`)
- WASM shaping backend (owned by `@paragraf/shaping-wasm`)
- Rendering (owned by `@paragraf/render-core`, `@paragraf/render-pdf`)

## Process Rules

- TDD is mandatory — tests before tasks, no exceptions unless explicitly overridden for a specific workId
- Division of labour: test descriptions are human-authored; test code and task implementations are LLM-generated
- Cache config parameters are user-configurable — never hardcoded
- `FontEngine` is a pluggable interface — implementations (`FontkitEngine`, `WasmFontEngine`) are interchangeable at the caller
- Layer L1: may import from L0 only (`@paragraf/types`, `@paragraf/color`). Never import from L2+.

---

## Consistency Checks (Copilot-executed)

Pre-task:
- [ ] Active plan workIds exist in `docs/work-pool.md`
- [ ] io-schema.md types match exported TypeScript types in `src/index.ts`
- [ ] dependency.md matches `1b-font-engine/package.json` dependencies

Post-task:
- [ ] io-schema.md updated if public types changed
- [ ] io-schema.md changes flagged for sync to project-level `docs/io-schemas.md`
- [ ] dependency.md updated if imports changed
- [ ] dependency.md changes flagged for sync to project-level `docs/dependency.md`
