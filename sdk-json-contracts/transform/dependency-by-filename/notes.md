# dependency-by-filename

**Pins:** `STD-WIRE-020`  ·  **Status:** OPEN
**Derived from:** DOC: cc-home/docs/06 'Dependencies are defined as the file name of the dependent compute manifest'

Edges in the DAG are identified by **filesystem path**. Two manifests with the same
 `manifest_name` in different directories are distinct nodes; moving a file silently rewires
 the graph with no change to the DAG's meaning and no diff that a reviewer would catch.

Open question for the ADR: the core library must canonicalize these paths before comparing
them, or `./first.json`, `first.json` and `../a/first.json` can name the same node three ways
and produce either a duplicate node or a missed edge. Nothing here is testable in an SDK
alone, which is why it is recorded as a transform fixture rather than a payload fixture.
