---
type: data
version: 3
updated: 260503-1730
actions: [read, append, update]
frequency: [on-demand, per-session]
---

# props

| Field            | Values                                                                     | Purpose                        |
| ---------------- | -------------------------------------------------------------------------- | ------------------------------ |
| `workId`         | integer                                                                    | unique identifier, never reused |
| `type`           | `spec` \| `issue-bucket` \| `package`                                     | work category                  |
| `title`          | string                                                                     | one-line description           |
| `status`         | `draft` → `planned` → `in-progress` → `human-review` → `done` \| `deferred` \| `cancelled` | current state |
| `packages`       | list                                                                       | affected packages              |
| `target-version` | semver or empty                                                            | estimated release              |
| `target-date`    | YYMMDD or empty                                                            | committed delivery             |
| `created`        | YYMMDD-HHMM                                                                | creation timestamp             |
| `updated`        | YYMMDD-HHMM                                                                | last update timestamp          |
| `findings`       | list of F-IDs or `—`                                                       | constituent findings (issue-bucket only) |
---

# rules

- `done` is terminal — no transitions out
- Status transitions are explicit human actions — plan creation alone does not advance `draft` → `planned`
- `deferred` reachable only from `in-progress`
- Completed and cancelled entries move to archive at outer-loop review gate
- A workId is assigned at creation and never reused

---

# entries

| workId | type | title | status | packages | findings | target-version | target-date | created | updated |
|--------|------|-------|--------|----------|----------|----------------|-------------|---------|---------|
| 002 | spec | log mechanism | draft | tbd | — | — | — | 260415-1430 | 260415-1430 |


---

# state transitions

```
draft → planned → in-progress → done
                              → deferred → in-progress (resume)
       → cancelled (from any state except done)
```

`done` is terminal — no transitions out.