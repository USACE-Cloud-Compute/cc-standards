# recursive-token

**Pins:** `STD-WIRE-010 / STD-WIRE-011`  ·  **Status:** OPEN
**Derived from:** CODE: regex `[^{}]*` cannot match a value containing braces, so `has{braces}inside` is not resolvable as a token

Attribute `inner` is `has{braces}inside`. Two-stage question: does the SDK (a) substitute once and leave the braces, producing `has{braces}inside/out.hdf`, or (b) re-scan and treat the embedded `{braces}` as a second token, which is recursive expansion?

**Recursive expansion is a security problem, not just a semantics one** (STD-SEC-020 / STD-WIRE-012): an attribute value that reaches the payload from a project-level input could name a variable the plugin was never granted, and a two-pass expansion makes that reachable. Recommend single-pass with residual braces rejected as an error. Whichever is chosen, all four SDKs must match.
