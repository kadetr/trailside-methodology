---
type: data
scope: project
tags:
  - data
  - work-pool
updated: 260429-1200
actions:
  - read
  - append
  - update
frequency:
  - on-demand
  - per-session

---

# entries

| workId | type | title | status | packages | findings | target-version | target-date | created | updated |
|--------|------|-------|--------|----------|----------|----------------|-------------|---------|---------|
| 014 | spec | PDF/X-3 metadata compliance | done  | render-pdf, compile | — | — | — | 260419 | 260430 |
| 006 | spec | cache key registry identity | done | typography | — | — | — | 260416-0100 | 260430-1400 |
| 004 | spec | featureSetId in @paragraf/style | done | style | — | — | — | 260415-1430 | 260430-1800 |
| 003 | spec | multi-column frame layout | done | layout, template | — | — | — | 260415-1430 | 260501-0900 |
| 101 | issue-bucket | Typography composition correctness and singleton reliability | done | typography, compile | — | — | — | 260501-1100 | 260501-1400 |
| 010 | issue-bucket | Fix silent-failure for spaceBefore/spaceAfter and hyphenation:false in compile pipeline | done | style, compile | — | — | — | 260501-1200 | 260501-1215 |
| 102 | issue-bucket | Fix singleLinePenalty condition, JSDoc, and add correctness test | done | linebreak | — | — | — | 260502-1100 | 260502-1100 |
| 103 | issue-bucket | Resolve compile.ts technical-debt: overflow reimplementation and missing deriveLineWidths call | done | compile | — | — | — | 260502-1200 | 260502-1200 |
| 104 | issue-bucket | Fix WASM ratio precision clamp in shaping-wasm | done | shaping-wasm | — | — | — | 260502-1100 | 260502-1130 |
| 108 | issue-bucket | Bucket D — Correctness: color + render-pdf | done | color, render-pdf | F009, F016 | v0.6.1 | — | 260503-1430 | 260503-1630 |
| 105 | issue-bucket | Bucket A — API surface cleanup | done | types, typography, shaping-wasm, font-engine | F010, F011, F012 | v0.6.1 | — | 260503-1430 | 260503-1553 |
| 107 | issue-bucket | Bucket C — Test coverage: color + render | done | color, render-core, render-pdf, template | F019, F021, F022, F023 | v0.6.1 | — | 260503-1430 | 260503-1700 |
| 106 | issue-bucket | Bucket B — Test coverage: typography | done | typography | F014, F018 | v0.6.1 | — | 260503-1430 | 260503-1730 |
| 109 | issue-bucket | Bucket E — Silent-failure + latent-bug | done | types, compile, render-core | F008, F017 | v0.6.1 | — | 260503-1430 | 260503-1800 |
| 114 | issue-bucket | Bucket J — Feature: style + template | done | style, template | F031, F032 | v0.6.1 | — | 260503-1430 | 260503-1830 |
| 110 | issue-bucket | Bucket F — Correctness: BiDi typography | done | typography | F013 | v0.6.1 | — | 260503-1430 | 260503-1900 |
| 112 | issue-bucket | Bucket H — Feature: linebreak + shaping | done | linebreak, shaping-wasm, render-core | F025, F026, F028, F029 | v0.6.1 | — | 260503-1430 | 260503-2000 |
| 116 | package | oma style propagation | done | style, compile | — | v0.6.1 | — | 260508-1600 | 260508-1600 |
| 117 | package | studio schema translation layer | done | studio | — | v0.7.0 | — | 260416-2230 | 260508-1900 |
| 118 | package | studio electron shell ipc | done | studio | — | — | — | 260508-2330 | 260508-2345 |
| 119 | package | studio preview panel frame overlay compile settings | done | studio | — | — | — | 260508-1030 | 260509-0000 |

---

# state transitions

```
draft → planned → in-progress → done
                              → deferred → in-progress (resume)
       → cancelled (from any state except done)
```

`done` is terminal — no transitions out.

---

# backlog notes

(no entry yet)

---

# changelog

 (no entry yet)
| 111 | issue-bucket | Bucket G — Feature: compile pipeline | done | compile, typography | 260503-1430 | 260503-1430 | F027, F036 |
| 113 | issue-bucket | Bucket I — Feature: render-pdf | done | render-pdf | 260503-1430 | 260503-1430 | F034, F035 |
| 115 | issue-bucket | Bucket K — Feature: investigations | done | color, typography, shaping-wasm, render-pdf | 260503-1430 | 260503-2100 | F024, F030, F033, F037, F038 |
