# multipart-datasource

**Pins:** `STD-STR-006`  ·  **Status:** PINNED
**Derived from:** Multi-file DataSource staging requirement

Shapefiles are three or more files addressed as one DataSource. If the writer publishes `s.shp` then dies before `s.dbf`, the DAG marks the node complete on a successful `s.shp` write and a downstream plugin reads an attribute-less geometry — often without erroring at all.

Required behavior: stage all components, then publish, so a mid-write failure leaves **no** partial DataSource visible. This fixture asserts the failure case explicitly, because the success case looks fine either way.
