# unknown-keys

**Pins:** `STD-WIRE-003`  ·  **Status:** OPEN
**Derived from:** CODE: Go unmarshal into a struct discards untagged keys entirely

**The most important fixture in the suite, and it currently fails in Go by construction.** `json.Unmarshal` into `Payload` drops `futureScalar`, `futureTopLevelArray`, and the unknown key inside `stores[0]`. Go has no unknown-field preservation without an explicit `map[string]json.RawMessage` sink or a round-trip document type. So STD-WIRE-003 is a **proposed requirement that the reference implementation does not meet**.

The concrete corruption: a manifest written by a newer CLI, opened and re-saved by an older-SDK plugin, silently loses the newer fields. The job still starts. The failure surfaces later as missing data downstream in the DAG.

The suite must be run to establish which of the four SDKs preserve unknown keys today, and an ADR must then either fund the change (add a passthrough document to every model type) or formally weaken STD-WIRE-003. Both are defensible; not deciding is not.
