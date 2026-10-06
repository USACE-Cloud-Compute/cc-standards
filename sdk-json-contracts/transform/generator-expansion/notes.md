# generator-expansion

**Pins:** `STD-OPS-002 / STD-OPS-003`  ·  **Status:** PINNED
**Derived from:** DOC: cc-home/docs/07_compute-file.md generator section

**Cost, stated as a contract.** `(end - start + 1) * len(perEventLoop)` jobs: this
 manifest is 2000 x 2 = **4000** jobs, not 2000. The documented Trinity example is exactly
 this shape.

Any tool that estimates a run MUST multiply by `perEventLoop` length. A reviewer approving a
change from `perEventLoop: [a]` to `[a,b,c,d]` is approving a 4x bill, and in a diff that
change reads as one added line. Also note `perEventLoop` values are strings in the docs
(`"DEBUG_CELL": "1"`) even where the meaning is boolean — the same stringly-typed hazard as
`compute_environment.vcpu`.

`CC_EVENT_NUMBER` is what output paths substitute on, so a per-event-loop run writing to a
path that does not include it will have every iteration collide (STD-OPS-005).
