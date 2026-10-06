# missing-object

**Pins:** `STD-PLG-006 / STD-PLG-011`  ·  **Status:** PINNED
**Derived from:** StoreReader.Get on an absent key

Absent must be a distinct, named outcome — not an empty read, not a zero-length success. An empty read that returns a valid empty stream lets the plugin compute a plausible zero, which in a flood-damage rollup is indistinguishable from 'no damage'. Maps to exit code 3 (input error, non-retryable): retrying an absent object 100 times wastes a fleet's worth of money.
