# outputs-not-readable

**Pins:** `STD-PLG-006`  ·  **Status:** PINNED
**Derived from:** CODE: Put()/Copy() resolve via GetOutputDataSource; GetReader() via GetInputDataSource

`Put` and `Copy`'s destination lookup go through `GetOutputDataSource` (outputs only), while `GetReader` and `Copy`'s *source* lookup also use `GetOutputDataSource` — an apparent bug in `Copy`, which cannot copy from an input. Either way the asymmetry is observable: a DataSource present in the payload is not necessarily readable.

Worth a fixture because the answer is not obvious from the API names, and `Copy` failing on an input source will surprise anyone who reads its signature. Confirm the `Copy` source-lookup behavior in each SDK and file a bug where it differs.
