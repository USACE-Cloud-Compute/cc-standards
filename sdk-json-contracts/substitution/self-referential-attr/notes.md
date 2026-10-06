# self-referential-attr

**Pins:** `STD-WIRE-010 / STD-WIRE-011`  ·  **Status:** OPEN
**Derived from:** CODE: attribute `self` is literally `{ATTR::self}`

The attribute value *is* a token naming itself. Single-pass leaves `{ATTR::self}/out.hdf` unresolved; a loop-until-stable implementation never terminates. Pinned because the failure must be a bounded, named error rather than a hang — a hung plugin inside a container that has already been billed for its memory reservation is the worst way to discover this (STD-PLG-013).
