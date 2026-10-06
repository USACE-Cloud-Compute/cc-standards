# naming-collisions

**Pins:** `STD-WIRE-005`  ·  **Status:** PINNED
**Derived from:** Field names as documented; collision is inherent to the two shapes

`store_name` and `profile` are structural keys on a DataStore, and they are also legal
 payload **attribute names**. Here both appear inside `payload_attributes`, where they are
 ordinary strings with no structural meaning.

Pinned because a "helpful" validator or SDK that treats any `store_name` key as a store
reference, or an SDK that recursively rewrites structural keys, will corrupt attribute data.
The rule is positional, not lexical: `store_name` is structural only at
`inputs[].store_name` / `outputs[].store_name`. Anywhere else it is data.
