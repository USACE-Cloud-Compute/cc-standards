# listing-order

**Pins:** `STD-STR-003`  ·  **Status:** PINNED
**Derived from:** Object-store listing is not sorted; S3 returns keys in UTF-8 binary order

Listing order is **byte order**, not human order: `10.json` before `2.json`, uppercase before lowercase. `STD-STR-003` forbids assuming a listing is sorted, and this fixture shows why the assumption looks safe and isn't — a developer's three-file local test always appears sorted.

Consequence for aggregation: a plugin that sums a listing in returned order gets a different float sum depending on backend and locale (STD-DET-002). Sort explicitly by key, in code, in every plugin.
