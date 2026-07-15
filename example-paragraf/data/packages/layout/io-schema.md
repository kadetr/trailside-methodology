# layout — I/O Schema

---
type: io-schema
package: @paragraf/layout
updated: 260501-0900
---

## Purpose

All public types, interfaces, classes, and functions exported by `@paragraf/layout`.

Types consumed from `@paragraf/types` but not re-exported: `Frame`, `BaselineGrid`.

---

## Unit Converters

All functions return **points (pt)**. 1 inch = 72pt. 1mm ≈ 2.8346pt.

### `mm(value: number): number`
Millimetres → points.

### `cm(value: number): number`
Centimetres → points.

### `inch(value: number): number`
Inches → points.

### `px(value: number, dpi?: number): number`
Pixels → points. `dpi` defaults to 96 (CSS pixel).

### `Dimension`
```ts
type Dimension = number | string
```
A dimension value: either a `number` (already in points) or a string with a unit suffix: `'20mm'`, `'2cm'`, `'0.5in'`, `'36pt'`, `'100px'`.

### `parseDimension(d: Dimension): number`
Resolve a `Dimension` to points. Throws on unrecognised string format.

---

## Page Sizes

### `PAGE_SIZES`
```ts
const PAGE_SIZES: {
  // ISO A-series
  A0: [number, number]; A1: [number, number]; A2: [number, number]
  A3: [number, number]; A4: [number, number]; A5: [number, number]; A6: [number, number]
  // ISO B-series
  B4: [number, number]; B5: [number, number]
  // SRA (bleed paper)
  SRA3: [number, number]; SRA4: [number, number]
  // North American
  Letter: [number, number]; Legal: [number, number]; Tabloid: [number, number]
}
```
All values are `[width, height]` in points in portrait orientation.

### `PageSizeName`
```ts
type PageSizeName = keyof typeof PAGE_SIZES
```

### `PageSize`
```ts
type PageSize = PageSizeName | [number, number]
```
Named size string or explicit `[width, height]` tuple in points.

### `resolvePageSize(size: PageSize): [number, number]`
Resolve a `PageSize` to a concrete `[width, height]` tuple. Throws on unknown named size.

### `landscape(size: PageSize): [number, number]`
Return the size in landscape orientation (wider side as width). No-op if already landscape.

### `portrait(size: PageSize): [number, number]`
Return the size in portrait orientation (taller side as height). No-op if already portrait.

---

## Page Layout

### `Margins`
```ts
interface Margins {
  top: number
  right: number
  bottom: number
  left: number
}
```
All values in points.

### `PageLayoutOptions`
```ts
interface PageLayoutOptions {
  size: PageSize
  margins: number | Margins   // single number = equal margins on all sides
  columns?: number            // default 1
  gutter?: number             // space between columns in points; default 0
  bleed?: number              // expanded on all four sides; default 0
}
```

### `Rect`
```ts
interface Rect {
  x: number
  y: number
  width: number
  height: number
}
```
Axis-aligned rectangle in points.

### `PageLayout`
```ts
class PageLayout {
  constructor(opts: PageLayoutOptions)   // throws on invalid bleed, columns, margins, gutter

  get pageSize(): [number, number]   // trim + 2×bleed on each axis; use for PDF MediaBox
  get trimSize(): [number, number]   // nominal/finished page size without bleed
  get trimBox(): Rect                // finished page boundary within bleed-expanded coordinate space
  get bleedBox(): Rect               // full bleed rectangle: { x: 0, y: 0, width: pageSize[0], height: pageSize[1] }

  frames(pageCount: number): Frame[] // one Frame per page, filling the printable area
}
```

Coordinate system: origin at top-left of the bleed-expanded page. Text frames are positioned relative to the trim edge (offset by `bleed + margin`).

Validation at construction: throws if margins exceed page size, gutter exceeds text area, bleed < 0, columns < 1.

### `columnWidths(frame: Frame): number[]`
Compute the width of each text column within a frame.
- Single-column frame: returns `[frame.width]`.
- Multi-column frame: `(frame.width - (n-1) × gutter) / n` for each of `n` columns.

---

## Region Layout

### `RegionSpec`
```ts
interface RegionSpec {
  height: Dimension    // required
  columns?: number     // default 1
  gutter?: Dimension   // between columns; default 0
  x?: Dimension        // offset from left of text area; default 0
  y?: Dimension        // offset from top of text area; omit for auto-stack
  width?: Dimension    // default: full text-area width
}
```
A rectangular area on a page, sub-divided into one or more columns.

### `framesForRegions(regions, textX, textY, textWidth, page): Frame[]`
```ts
framesForRegions(
  regions: RegionSpec[],
  textX: number,
  textY: number,
  textWidth: number,
  page: number,
): Frame[]
```
Produce one `Frame` per column of each region, in reading order
(all columns of region[0], then region[1], etc.).

- **Auto-stack**: when `y` is omitted, each region is positioned immediately below the previous one.
- **Explicit y**: when set, used as-is; the stack pointer still advances by the region's height.
- **Output frames** are single-column (`columnCount`/`gutter` are not set on output frames).
- `textX`/`textY` are the origin of the text area within the page coordinate space (bleed + margin).
