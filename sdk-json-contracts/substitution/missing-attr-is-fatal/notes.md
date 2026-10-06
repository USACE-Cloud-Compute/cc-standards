# missing-attr-is-fatal

**Pins:** `STD-WIRE-010 / STD-WIRE-011`  ·  **Status:** PINNED
**Derived from:** DOC: Confluence — 'If an attribute name or a environment variable is used in substitution but not provided at compute time, the PluginManager will fatally fail'

Documented as fatal. Fatal means **non-zero exit with the attribute name and the containing field in the message** (STD-PLG-011 code 2, STD-WIRE-011). An SDK that logs and continues has turned a startup error into a wrong answer.
