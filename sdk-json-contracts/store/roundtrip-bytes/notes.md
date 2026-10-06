# roundtrip-bytes

**Pins:** `STD-DET-008 / STD-STR-004`  ·  **Status:** PINNED
**Derived from:** Streaming read/write through StoreReader/StoreWriter

Binary fidelity through the store abstraction: HDF, DSS, and shapefile bytes must round-trip unchanged. The failure mode is a text-mode or encoding-transforming read, which corrupts an HDF5 file while reporting success. Also the guard against an SDK that buffers whole objects into memory (STD-STR-004 forbids it) — assert with a fixture larger than a declared streaming threshold, not just a small one.
