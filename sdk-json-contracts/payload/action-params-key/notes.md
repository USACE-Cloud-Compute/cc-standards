# action-params-key

**Pins:** `STD-SDK-003`  ·  **Status:** OPEN
**Derived from:** DOC: Confluence payload example uses `params`; CODE: Action embeds IOManager whose attributes tag is `attributes`

The Confluence example authors an action's parameters as `params`. `Action` in Go embeds `IOManager`, whose only attribute field is tagged `attributes` — there is **no `params` field on Action**, so Go drops those values and the action runs with empty attributes.

Expected model shows the union: whichever key an SDK reads, the *effective* attribute set must be identical. Recommended resolution is that `attributes` is canonical and `params` is accepted as a deprecated alias (STD-VER-008), because manifests may already be in the field written from the docs — and those manifests are currently being silently ignored by at least one SDK.
