# missing-name

**Pins:** `STD-WIRE-021`  ·  **Status:** PINNED
**Derived from:** Schema: data-store.name and action.name are required

Two nameless objects in one document, so the runner must prove the validator reports **every** violation with a JSON pointer, not just the first. Error messages that name only the first failure cost one round-trip per mistake in a manifest with several, and manifests are hand-edited.
