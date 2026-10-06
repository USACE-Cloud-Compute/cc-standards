# unknown-top-level-key

**Pins:** `STD-WIRE-002 / STD-WIRE-003`  ·  **Status:** OPEN
**Derived from:** Schema: additionalProperties true at payload root

Split deliberately from `payload/unknown-keys`: this one pins only that an unknown key **does not invalidate** the document. Strictness at the schema level and preservation at the parse level are independent decisions, and conflating them means a fix to one hides the other. If a future schema revision adds `additionalProperties: false`, it authorizes every SDK to reject forward-compatible manifests — which is the corruption vector STD-WIRE-003 exists to close.
