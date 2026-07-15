# work-pool Archive

---
type: data
version: x.y
updated: YYMMDD-HHMMSS
actions: [read, append, update]

---

# entries

| workId | type | title | status | packages | findings | target-version | target-date | created | updated |
|--------|------|-------|--------|----------|----------|----------------|-------------|---------|---------|


---

# state transitions

```
draft → planned → in-progress → done
                              → deferred → in-progress (resume)
       → cancelled (from any state except done)
```

`done` is terminal — no transitions out.
