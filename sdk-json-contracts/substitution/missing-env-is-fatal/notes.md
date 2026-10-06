# missing-env-is-fatal

**Pins:** `STD-WIRE-010 / STD-WIRE-011`  ·  **Status:** PINNED
**Derived from:** DOC: same as missing-attr-is-fatal

Same rule, environment side. Distinct error class on purpose: the operator's remediation differs completely (fix the manifest vs fix the compute environment), and a single generic 'substitution failed' message costs a full scheduling round-trip to diagnose.
