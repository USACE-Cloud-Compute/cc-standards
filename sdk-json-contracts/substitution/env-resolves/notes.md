# env-resolves

**Pins:** `STD-WIRE-010 / STD-WIRE-011`  ·  **Status:** PINNED
**Derived from:** CODE: CcEventNumber = "CC_EVENT_NUMBER"; DOC: Confluence 'Payload Substitutions'

Environment substitution. `CC_EVENT_NUMBER` is what segregates outputs across a parallel run — if it fails to resolve, every event in a 10,000-event batch writes to the **same key** and the last writer wins. That is silent data loss at scale, which is why the empty-vs-missing distinction below matters more than it looks.
