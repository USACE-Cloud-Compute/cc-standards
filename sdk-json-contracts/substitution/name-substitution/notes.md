# name-substitution

**Pins:** `STD-WIRE-010 / STD-WIRE-011`  ·  **Status:** PINNED
**Derived from:** DOC: Confluence — substitution applies to 'DataSource Names, Paths, and DataPaths'

Substitution in the DataSource **name**, not just paths — which changes the key a plugin must use for lookup. Pinned because a name-substituting SDK and a path-only-substituting SDK disagree about whether `get_input_data_source("{ATTR::modelPrefix}.hdf")` or `get_input_data_source("bardwell-creek.hdf")` finds it. The docs also carry 'they can occur in Attributes????' with the question marks still in the published page, so attribute-value substitution is formally unresolved.
