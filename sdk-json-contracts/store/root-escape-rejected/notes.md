# root-escape-rejected

**Pins:** `STD-SEC-020 / STD-STR-005`  ·  **Status:** OPEN
**Derived from:** Required behavior; NOT currently implemented in cc-go-sdk GetAbsolutePath

**This is a security requirement the reference implementation does not enforce.** `GetAbsolutePath` does `filepath.Clean(root + sep + path)` and returns the result with no containment check, so `../../etc/passwd` cleans to a path **outside** `params.root`. Combined with `{ATTR::}` substitution inside `paths`, a payload whose attribute values are influenced by project-level input controls where a plugin reads and writes.

Marked OPEN because adding the check is a behavior change that could break a manifest that legitimately walks above its declared root — but the fix direction is not optional: normalize, verify containment, reject `..`, absolute paths, embedded NULs and symlink escapes. Every SDK needs the same check, and this fixture is what proves each one has it.
