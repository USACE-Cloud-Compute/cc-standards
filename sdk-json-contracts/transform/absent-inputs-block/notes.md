# absent-inputs-block

**Pins:** `STD-PLG-006`  ·  **Status:** OPEN
**Derived from:** Inferred: the compute manifest's `inputs` block is optional per the cc-home field table

A compute manifest with **no `inputs` block at all** must produce a payload with
 `"inputs": []` and `"attributes": {}` — not `null`, and not the keys omitted. This is where
 the absent-vs-empty distinction from `payload/minimal` actually gets exercised, because the
 authoring shape permits omission while the runtime shape is expected to be uniform.

If the core library emits `"attributes": null` instead of `{}`, a Go SDK unmarshals nil
`PayloadAttributes` and every `Get*` call returns "Attribute X is not in the payload" rather
than a distinguishable 'no attributes were supplied'. Worth pinning before four SDKs each
choose. Note also `inputs.data_sources` vs payload `inputs`: an author can supply
`inputs.environment` with no `data_sources`, which must still yield `inputs: []`.
