# bad-store-type

**Pins:** `STD-STR-007`  ·  **Status:** PINNED
**Derived from:** CODE: StoreType constants S3/FS/WS/RDBMS/EBS; registry holds only S3 and FS

`S4` fails the enum. The interesting case is the one this fixture does **not** cover: `WS`, `RDBMS` and `EBS` are valid enum members that `registerStoreTypes()` never registers, so they pass schema validation and then die at `connectStores` with 'Unregistered store type'. A validator that stops at the schema tells the author the manifest is fine and the failure arrives a full scheduling round-trip later. SDKs SHOULD warn on a schema-valid but unregistered backend.
