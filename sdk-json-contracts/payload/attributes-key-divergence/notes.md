# attributes-key-divergence

**Pins:** `STD-WIRE-005 / STD-SDK-001`  ·  **Status:** OPEN
**Derived from:** CODE: `json:"attributes"` in payload.go vs DOC: `payloadAttributes` in the Confluence payload example

**A live defect, not a hypothetical.** Two authoritative sources name the same field differently:

| Source | Key |
|---|---|
| `cc-go-sdk` struct tag (what actually serializes) | `attributes` |
| Confluence SDK page + payload example | `payloadAttributes` |

An SDK written from the docs reads `payloadAttributes`, finds nothing, and gets an **empty attribute map**. Every `{ATTR::}` substitution then either fatally fails (correct-ish, loud) or resolves to empty string (silent wrong paths — the dangerous one).

This fixture puts BOTH keys in one document with DIFFERENT values so the runner can tell which one an SDK actually read, from a single parse. The expected model follows the code, because the code is what produced the payloads already sitting in customer stores.

**Action:** confirm against `cc-py-sdk`/`cc-java-sdk`/`cc-dotnet-sdk` parsers, then an ADR picks one key, deprecates the other under STD-VER-008, and the docs get corrected. Until then plugin authors must be told which key to write.
