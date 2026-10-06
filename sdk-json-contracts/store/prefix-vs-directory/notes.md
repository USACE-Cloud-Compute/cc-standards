# prefix-vs-directory

**Pins:** `STD-STR-002`  ·  **Status:** PINNED
**Derived from:** Object store prefix semantics vs filesystem directory semantics

The single most common FS/S3 divergence. On a filesystem, `root=mod` means a directory and `modother` is unreachable; on S3 `mod` is a raw key prefix and `modother/y.json` **matches**. A plugin validated only against `FS` under-counts or over-counts inputs when it runs against `S3` in production, and the count is the answer in a damage rollup.

Rule for plugin authors: always treat a prefix as ending in a separator, i.e. list `mod/` not `mod`. CI must run storage tests against both backends (STD-STR-002).
