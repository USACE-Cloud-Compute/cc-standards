# duplicate-datasource-name

**Pins:** `STD-WIRE-021`  ·  **Status:** OPEN
**Derived from:** CODE: IOManager.GetDataSource returns on first name match; no duplicate check exists

Go returns the **first** match and never detects the duplicate, so a payload with two `model` DataSources quietly uses `a/model.json` while a plugin author believes they wrote `b/`. Nothing in the current code rejects this.

Undocumented either way, and it is exactly the class of bug that produces a correct-looking answer from the wrong input. Recommend the ADR pick *error at load*: it is the only option that cannot produce a silently wrong FFRD result, and a payload with duplicate names is almost certainly an authoring mistake rather than intent.
