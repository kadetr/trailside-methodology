---
type: data
scope: project
tags:
  - data
  - architecture
  - dependency
  - io-schemas
updated: [YYMMDD-HHMMSS]
actions:
  - read
  - update
frequency:
  - on-demand
  - per-session
---

# props — architecture (outer loop)

| Field        | Values                                      | Purpose                                               |
| ------------ | ------------------------------------------- | ------------------------------------------------------ |
| `package`    | string                                      | short package name                                    |
| `layer`      | [project layer scheme, e.g. 0–4 \| App]     | architectural layer identifier                        |
| `dependency` | list                                        | key internal architectural dependencies (not exhaustive) |
| `status`     | `stable` \| `experimental` \| `deprecated`  | package maturity                                      |
| `third-party-deps` | list                                  | third-party dependencies                              |

# props — io-schemas (inner loop / on-demand)

| Field         | Values       | Purpose                                  |
| ------------- | ------------ | ------------------------------------------ |
| `layer`       | [layer scheme] | architectural layer                      |
| `npm-name`    | string       | full package name                        |
| `exports`     | string       | summary of what the package exports      |
| `schema-file` | path         | link to per-package io-schema.md         |

# rules

- Architecture section used at outer loop / session boundary — layer and structural deps
- Dependency section used during inner loops — full runtime deps per package
- IO-schemas section is a navigation index only — full definitions live in per-package io-schema.md
- Dev dependencies not listed here — see project's own dependency manifest
- Optional deps noted with a symbol in the table (e.g. ‡ for graceful fallback)
- Updated when a package is added, removed, its layer changes, its imports change, or its exported types change

---

# architecture & dependency — package layer map

| package | layer | dependency | status | third-party-deps |
|---|---|---|---|---|
| [package-name] | [layer] | [deps or —] | [stable/experimental/deprecated] | [deps or —] |

<uncertain class="stub">Table emptied to one placeholder row. Source had 14 populated rows describing paragraf's actual dependency graph — genuinely project-specific data, not shape. Fill in your own architecture here.</uncertain>

‡ optional / graceful-fallback dependency

---

# io-schemas — navigation index

| layer | npm-name | exports | schema-file |
|---|---|---|---|
| [layer] | [package-name] | [what it exports] | data/packages/[package]/io-schema.md |

<uncertain class="stub">Same as above — emptied to one placeholder row.</uncertain>

---

# layer rules

<uncertain class="stub">Source defined five concrete layers (L0 foundational types, L1 primitives, L2 accelerated/WASM backends, L3 composition/rendering, L4 orchestration, App application-tier) with specific import-direction rules for a rendering-library architecture. This is real architectural policy, not generic shape — kept as an illustrative example below, but you should replace it with your own project's layering scheme rather than adapt it in place.</uncertain>

- **[Layer 0] — foundational primitives.** [Import/dependency rule.]
- **[Layer 1] — ...**
- ...
- **A package may NOT import from a higher layer than its own.** [State your project's actual layering discipline here.]
