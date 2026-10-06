# partial-token-literal

**Pins:** `STD-WIRE-010 / STD-WIRE-011`  ·  **Status:** PINNED
**Derived from:** CODE: regex applied with ReplaceAllString, not full-match

A token embedded in a longer string must substitute in place. An SDK that treats the field as a token only when the *whole* value matches (a common misreading) leaves this path unresolved and the plugin reads a literal `prefix-{ATTR::modelPrefix}-suffix.hdf`.
