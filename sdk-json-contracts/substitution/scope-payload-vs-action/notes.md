# scope-payload-vs-action

**Pins:** `STD-WIRE-010 / STD-WIRE-011`  ·  **Status:** PINNED
**Derived from:** CODE: substituteMapVariables(pm.Attributes, **false**) for payload attributes vs **(action.Attributes, true)** for action attributes

**An asymmetry in the reference implementation that is almost certainly unintended.** The second argument differing between the two call sites means action attributes and payload attributes do not substitute the same way. This fixture is the payload-scope case; `scope-action-attributes` is the action case. Run both and compare: if a value containing `{ENV::}` behaves differently depending on which scope it sits in, that is a bug to file, not a feature to standardize. Recording the current asymmetry as PINNED for the payload side because it follows the documented 'attributes substitute ENV' behavior only at action scope.
