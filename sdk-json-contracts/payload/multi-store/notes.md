# multi-store

**Pins:** `STD-STR-002 / STD-STR-007`  ·  **Status:** PINNED
**Derived from:** DOC: cc-home 06 stores sample (verbatim shape)

Two stores with different `store_type` and different connection requirements in one payload: S3 needs a `profile` and opens a client, FS does not. `FS` here has no `profile` key at all, which is correct and documented. The pin is that `store_name` resolves by exact string match against the *declared* stores — a typo is a hard error, not a fallback to a default store.
