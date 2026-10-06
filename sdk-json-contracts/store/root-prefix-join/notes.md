# root-prefix-join

**Pins:** `STD-STR-005`  ·  **Status:** PINNED
**Derived from:** CODE: GetAbsolutePath -> filepath.Clean(fmt.Sprintf("%s%c%s", root, os.PathSeparator, path))

How `params.root` joins to a DataSource path, including the `filepath.Clean` step. Pinned because the naive join variants are all broken in different ways: no separator gives `/daroot/a/b.json`, a doubled separator gives `//data/root//a/b.json`, and a DataSource path that is *absolute* (`/etc/passwd`) escapes the root entirely — which is the STD-SEC-020 traversal check this fixture exists to exercise.
