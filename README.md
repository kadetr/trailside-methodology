# TRAILSIDE METHODOLOGY

A four-layer execution architecture for AI-assisted development — implemented
entirely in markdown, with no runtime or framework dependency.

Most delegation frameworks tell an LLM what to do. This one is built around a
narrower question: **where, structurally, is an LLM allowed to make a
judgment call?** The answer here is: in exactly one layer, through exactly
one primitive. Everywhere else, branching is legal only on data — a file
value, a status field — never on interpretation. That constraint is enforced
by the shape of the files themselves, not by asking the model nicely.

This repo contains two things:

- **`template/`** — an empty skeleton. Copy it into a new project and fill in
  the blanks (marked inline) to adopt the methodology.
- **`example-paragraf/`** — a real instance, pulled from an active project
  (`paragraf`, a PDF/typography rendering library) after months of use. Not
  a demo — the actual working state, including the rough edges.

If you want the reasoning behind the design, not just the artifact, see the
companion write-up: **[A Practitioner's Account — Article III](#)**
*(link — insert dev.to URL)*.

---

## The permission model

Four layers, each with a fixed contract. This table is the entire security
model — everything else in the repo exists to enforce it.

| Layer | Condition (data) | Judgment (via `ASK`) | May call |
|---|---|---|---|
| Primitive | no | no | nothing |
| Sequence | no | no | primitives |
| Procedure | yes | no | primitives, sequences |
| Flow | yes | **yes — only here** | all layers |

A Procedure or Sequence that needs to make a judgment call isn't
under-specified — it's a design signal that the work belongs one layer up.

---

## Directory structure

```
template/                     ← empty skeleton — start here to adopt
├── index.md                  ← entry point: load order, layer contracts, ask-human triggers
├── primitives.md             ← the 9 verb primitives (READ, WRITE, ASK, ...) and their layer restrictions
├── sequences.md              ← named, silent primitive-compositions (no judgment, no state writes)
├── procedures/                ← linear, non-branching control flow (session open/close, archiving)
├── flows/                     ← branching, iterating, human-facing orchestration
├── data/                      ← project knowledge: work-pool, issues, architecture, per-package context
├── state/current.yaml         ← live session state — the only file written field-by-field, not wholesale
├── plans/, logs/, archive/    ← per-work artifacts and their eventual resting place

example-paragraf/             ← real instance — same shape, populated
├── ...                        ← identical structure, filled with actual project data
└── archive/old-files/         ← deprecated flows kept for history (f-start, f-group-issues)
```

Both trees have the same shape. That's deliberate — the template *is* the
instance with the project-specific content removed, not a separate document
that happens to describe it.

---

## Reading order

1. **`index.md`** in either tree — the entry point every session starts
   from. It defines load order, the layer contracts above, and the
   ask-human trigger table.
2. **`primitives.md`** → **`sequences.md`** — the full verb vocabulary,
   read once per session.
3. **`flows/f-work-run.md`** — the main session lifecycle, if you want to
   see the layers actually composed end-to-end.
4. **`data/work-pool.md`**, **`data/issues.md`** — how work is tracked and
   how review findings become planned work.

## Adopting the template

1. Copy `template/` into your project.
2. Fill in `data/architecture.md`, `data/packages.md`, and one
   `data/packages/[name]/` pair per architectural unit.
3. Add your first entry to `data/work-pool.md`.
4. Run `flows/f-work-run.md` with that `workId`.

Everything else — plan drafting, task execution, archiving — is handled by
the flows and procedures already in place.

## Status

This methodology is versioned alongside the project it governs. See
`index.md` frontmatter (`version:`) in each tree for the current revision.
Structural changes are handled as version boundaries with a handoff
document, not in-place refactors — see Article III §[N] for the rationale.

## License

*(TBD — CC BY 4.0 is a reasonable default for prose/spec content like this;
confirm before publishing)*
