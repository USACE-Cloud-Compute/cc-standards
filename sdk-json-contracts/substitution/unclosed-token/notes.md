# unclosed-token

**Pins:** `STD-WIRE-010 / STD-WIRE-011`  ·  **Status:** PINNED
**Derived from:** CODE: regex `{([^{}]*)}` requires a closing brace

A brace with no closer is not a token and must survive **literally**. Derivable from the regex. Pinned because a hand-rolled parser that splits on the first `{` will consume the remainder of the string and mangle the path.
