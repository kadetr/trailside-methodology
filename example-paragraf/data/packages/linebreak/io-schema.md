# linebreak — I/O Schema

---
type: io-schema
package: @paragraf/linebreak
updated: 260416-1930
---

## Purpose

All public types, interfaces, and functions exported by `@paragraf/linebreak`.

Types consumed from `@paragraf/types` but not re-exported: `Paragraph`, `Node`, `Box`, `Glue`, `Penalty`, `BreakpointNode`, `Font`, `SpanSegment`, `AlignmentMode`, `ComposedLine`, `ComposedParagraph`, `GetFontMetrics`, `Measurer`, `Language`.

---

## Algorithm Types

### `BreakpointResult`
```ts
interface BreakpointResult {
  node: BreakpointNode   // best final BreakpointNode from the forward pass
  usedEmergency: boolean // true if emergencyStretch was required to find a solution
}
```

### `LineBreak`
```ts
interface LineBreak {
  position: number   // index in node sequence where line ends
  ratio: number      // adjustment ratio — how much glue was stretched (>0) or shrunk (<0)
  flagged: boolean   // true if line ends with a hyphen
  line: number       // line number (1-based)
}
```

---

## Algorithm Functions

### `computeBreakpoints(paragraph: Paragraph): BreakpointResult`
Knuth-Plass forward pass. Finds the optimal set of breakpoints that minimises total demerits.
Throws if no feasible solution is found even with `emergencyStretch`.

Reads from `Paragraph`: `nodes`, `lineWidth`, `lineWidths`, `tolerance`, `emergencyStretch`, `consecutiveHyphenLimit`, `widowPenalty`, `orphanPenalty`, `looseness`.

### `traceback(finalNode: BreakpointNode): LineBreak[]`
Walk `previous` pointers from the final best `BreakpointNode` back to the start.
Returns `LineBreak[]` in document order (first line first).

### `composeParagraph(nodes: Node[], breaks: LineBreak[], alignment?: AlignmentMode, justifyLastLine?: boolean, lineWidth?: number, lineWidths?: number[], getMetrics?: GetFontMetrics, direction?: 'ltr' | 'rtl'): ComposedParagraph`
Convert a node sequence and resolved breakpoints into composed lines.

| Parameter | Default | Notes |
|---|---|---|
| `alignment` | `'justified'` | Applied to all lines; last line uses natural spacing unless `justifyLastLine` |
| `justifyLastLine` | `false` | If true, `lineWidth` must be provided |
| `lineWidth` | `0` | Used when `lineWidths` array doesn't cover a given line |
| `lineWidths` | `[]` | Per-line width overrides (0-indexed) |
| `getMetrics` | `undefined` | If provided, used to compute `lineHeight` and `baseline` per line |
| `direction` | `'ltr'` | Propagated to each `ComposedLine.direction` |

Returns `ComposedParagraph` (`ComposedLine[]`). Marks last single-word line as `isWidow: true`.

---

## Node Builder

### `HyphenatedWordWithFont`
```ts
interface HyphenatedWordWithFont extends HyphenatedWord {
  font: Font                  // dominant (first) font — used for single-font words
  segments?: SpanSegment[][]  // per-fragment span breakdown; present for multi-font words
}
```

### `buildNodeSequence(words: HyphenatedWordWithFont[], measurer: Measurer, firstLineIndent?: number): Node[]`
Build a Knuth-Plass node sequence from hyphenated words. Appends termination glue + forced-break penalty.
Handles single-font and multi-font (span segment) words. Uses `SOFT_HYPHEN_PENALTY` for user-specified soft hyphens, `HYPHEN_PENALTY` for algorithmic breaks.

---

## Hyphenation Types

### `HyphenateOptions`
```ts
interface HyphenateOptions {
  minWordLength: number          // skip words shorter than this
  fontSize: number               // used to derive minLeft / minRight if not overridden
  language: Language
  preserveSoftHyphens?: boolean  // default true — honour U+00AD in input
  minLeft?: number               // override deriveMinLeft(fontSize)
  minRight?: number              // override deriveMinRight(fontSize)
  processCapitalized?: boolean   // default false — skip non-first capitalized words (proper noun heuristic)
}
```

### `HyphenatedWord`
```ts
interface HyphenatedWord {
  original: string
  fragments: string[]    // split at hyphenation points
  hyphenable: boolean    // false if word was too short or language pattern produced no breaks
  hasSoftHyphen: boolean // true if fragments came from explicit U+00AD soft hyphens
}
```

### `DEFAULT_HYPHENATE_OPTIONS`
```ts
const DEFAULT_HYPHENATE_OPTIONS: HyphenateOptions = {
  minWordLength: 5,
  fontSize: 12,
  language: 'en-us',
  preserveSoftHyphens: true,
}
```

---

## Hyphenation Functions

### `loadHyphenator(language: Language): Promise<void>`
Load and cache the hyphenation pattern for `language`. Idempotent. Must be called before `hyphenateWord` / `hyphenateParagraph` for that language.

### `loadLanguages(languages: Language[]): Promise<void>`
Load multiple language patterns in parallel.

### `hyphenateWord(word: string, options: HyphenateOptions): HyphenatedWord`
Hyphenate a single word. Respects soft hyphens when `preserveSoftHyphens` is true. Returns the word unhyphenated if shorter than `minWordLength` or no pattern breaks are found.

### `hyphenateParagraph(words: string[], options: HyphenateOptions): HyphenatedWord[]`
Hyphenate all words in a paragraph. Applies proper-noun heuristic when `processCapitalized` is false.

### `deriveMinLeft(fontSize: number): number`
Derive minimum left fragment length from font size: `Math.max(2, Math.round(fontSize / 6))`.

### `deriveMinRight(fontSize: number): number`
Derive minimum right fragment length from font size: `Math.max(2, Math.round(fontSize / 6))`.

### `DEFAULT_HYPHENATE_OPTIONS: HyphenateOptions`
```ts
{ minWordLength: 5, fontSize: 12, language: 'en-us', preserveSoftHyphens: true }
```
Default options used when `hyphenateWord` / `hyphenateParagraph` are called without explicit options.

---

## Test Utilities

### `mockMeasure`, `mockSpace`, `mockMetrics`
Defined locally in `testing.ts`. Provide deterministic fixed-proportion measurement for unit tests. No fontkit dependency — safe to import anywhere.
