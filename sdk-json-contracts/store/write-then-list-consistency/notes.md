# write-then-list-consistency

**Pins:** `STD-STR-003`  ·  **Status:** PINNED
**Derived from:** Object store listing consistency after write

A newly written key may not appear in a listing immediately on every backend. Plugins that write then enumerate must not treat a missing listing entry as 'write failed' (that turns a retryable timing artifact into exit 3), and must not treat a stale listing as complete (that drops an output). Status is OPEN because it needs a decision about which side to favor; the recommendation is to make output publication depend on the write's own success, never on a subsequent listing.
