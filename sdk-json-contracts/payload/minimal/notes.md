# minimal

**Pins:** `STD-WIRE-021`  ·  **Status:** PINNED
**Derived from:** CODE: cc-go-sdk testdata/test_payload.json (verbatim)

The empty-payload baseline. `cc-go-sdk`'s own testdata uses empty arrays rather than omitting the keys, so **absent** and **empty** are distinct inputs an SDK must both accept. Go unmarshals a missing key to a nil slice and every consumer treats nil-as-empty, so the parsed models must be equal after normalization. An SDK whose strict parser rejects `{}` fails here.
