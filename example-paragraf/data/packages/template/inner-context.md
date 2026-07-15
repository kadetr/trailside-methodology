# template — Inner Context

---
type: inner-context
package: @paragraf/template
layer: L4
updated: 260501-0900
---

## Role

Document template schema definition and validation. Defines the data shape that users write to describe a document's layout, fonts, styles, and content slots. Provides `defineTemplate()` as the validation entry point and `parseTokens()` for interpolation syntax.

This package is schema-only — no composition, no rendering. It is consumed by `@paragraf/compile` which does the actual work.

---

## DSL Scope — intentional minimalism

The `{{binding.path}}` token syntax is intentionally minimal. There are no conditionals,
loops, filters, or format helpers. This is a design decision, not a gap.

**Rationale.** Formatting, conditional logic, and data transformation belong in the
caller's `normalize()` function, not in the template. This separation keeps:

- Templates static and validatable at construction time (`defineTemplate` can fully
  validate a template with no data).
- The template language readable to non-engineers: a template is a layout description,
  not a program.
- The compile pipeline simple and testable: every token resolves to a plain string.

**What to do instead of template-level logic:**

| Instead of… | Do this in `normalize(data)` |
|---|---|
| `{{?subtitle}}` conditional | Return `{ subtitle: '' }` and use `onMissing: 'hide'` on the slot |
| `{{price \| currency}}` filter | `return { price: formatCurrency(data.price) }` |
| `{{#each items}}` loop | Emit one content slot per item from `normalize`, or use a separate template per item |

**Future.** Optional shallow extensions (e.g. `{{?key}}` null-guard, `{{key \| filter}}`
named filters) may be added if the user demand is consistent — see #57. Any addition
must remain statically evaluable without a data record.

---



- `defineTemplate(input)` validates and returns the input unchanged. It is a pure validator: no mutations.
- Validation checks: style inheritance chains (no cycles, no missing `extends`/`next` refs), slot `style` references must exist in `template.styles`, `onMissing: 'fallback'` slots must have `fallbackText`, all `text` fields must parse without errors.
- `parseTokens(text)` parses `{{binding.path}}` syntax into `Token[]`. Throws on unclosed `{{`, empty path `{{}}`, or invalid dot-path (non-identifier segments).
- Binding paths: one or more dot-separated segments. First segment must be a non-numeric identifier. Subsequent segments may be identifiers or numeric indices (`items.0.price`).
- `TemplateLayout` uses `Dimension` strings throughout. Resolution to points happens in `@paragraf/compile`.
- `TemplateLayout.pages` (optional): array of `TemplatePageSpec` entries, each specifying a `range` (number, `'N+'`, `'N-M'`, or `'default'`) and an array of `TemplateRegionSpec` regions.
- `TemplateRegionSpec` mirrors `RegionSpec` from `@paragraf/layout` but uses `Dimension` strings. Fields: `height` (required), `columns?`, `gutter?`, `x?`, `y?`, `width?`.
- `TemplateLayout.columns` and `TemplateLayout.gutter` are **deprecated** as of workId 003. Use `pages` with `TemplateRegionSpec` instead. They remain in the schema for backward compatibility; removal is deferred to a future workId.
- `TemplateFontVariants` allows the four standard keys (`regular`, `bold`, `italic`, `boldItalic`) as string shorthand. Custom keys require the object form with explicit metadata.
- `ParagraphStyleDef` is re-exported from `@paragraf/style` so consumers can type their `styles` object without a separate import.
- `PageSize` and `Dimension` are re-exported from `@paragraf/layout`.

---

## Package Constraints

- **Layer L4**: imports from L0 (`@paragraf/types`), L1 (`@paragraf/layout`, `@paragraf/style`). Does **not** import from L2, L3, or any other L4 package.
- Zero side effects at import time. No module-level initialisation.
- `defineTemplate` returns the identical object reference it received (no clone/deepCopy).

---

## Consistency Checks

- Every key in `template.content[n].style` must exist in `template.styles`.
- Style `extends` and `next` references must form a DAG (no cycles).
- `onMissing: 'fallback'` requires `fallbackText !== undefined`.
