# action-name-vs-type

**Pins:** `STD-SDK-003`  ·  **Status:** OPEN
**Derived from:** CODE: pluginmanager.go RunActions() compares action.Name to registry keys; DOC: cc-py-sdk README `match action.type:`

The two SDKs **dispatch on different fields**. Go runs the runner registered under the action's `name`; the Python README pattern switches on `action.type`. This fixture deliberately gives every action a name that differs from its type, so a single run reveals which field an SDK dispatches on — with `prepare`/`solve` vs `run-steady`/`run-unsteady`, a Go plugin and a Python plugin reading the same payload execute *different code*.

This is the most severe parity finding in the set: it is not a formatting difference, it is the wrong computation running silently. Immediate guidance for plugin authors: **set `name` and `type` to the same value** until an ADR settles it. The ADR should probably pick `name`, because that is what the reference implementation and the registry model both use.
