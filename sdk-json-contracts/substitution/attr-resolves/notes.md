# attr-resolves

**Pins:** `STD-WIRE-010 / STD-WIRE-011`  ·  **Status:** PINNED
**Derived from:** CODE: substitutionRegexPattern = `{([^{}]*)}`; DOC: Confluence 'Payload Substitutions'

Baseline positive case: two tokens in one string, resolved from payload attributes. Also pins that `{ATTR::plan}` yields the **string** `"01"` — an SDK that coerces attributes to numbers produces `1` and a wrong path, which is the leading-zero hazard from `payload/full-attributes` showing up again.
