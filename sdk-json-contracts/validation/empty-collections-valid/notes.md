# empty-collections-valid

**Pins:** `STD-PLG-006`  ·  **Status:** PINNED
**Derived from:** CODE: cc-go-sdk testdata/test_payload.json

Empty is valid, and a plugin must tolerate it: a generator that produces zero events, or a payload assembled before the DAG is populated, both arrive like this. A plugin that indexes `outputs[0]` unconditionally panics on the first empty run. Cheap test, expensive production failure.
