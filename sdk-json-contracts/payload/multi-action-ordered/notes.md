# multi-action-ordered

**Pins:** `STD-WIRE-021`  ·  **Status:** PINNED
**Derived from:** DOC: cc-home SDK Concepts, 'Actions are performed in order ... can be repeated'

Array order is semantic and repetition is legal — the documented HEC-RAS example uses exactly this sequence, and a repeated action is how you run the same solver twice with different parameters. Pinned here because two implementations break it in opposite ways: a language whose JSON layer parses an array into an ordered map loses the duplicate, and one that sorts keys loses the order. Both produce a *plausible* run. The last two entries are identical in `type` and differ only in `name`, which is what makes the duplicate detectable.
