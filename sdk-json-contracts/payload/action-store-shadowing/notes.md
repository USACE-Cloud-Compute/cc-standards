# action-store-shadowing

**Pins:** `STD-SDK-001`  ·  **Status:** PINNED
**Derived from:** CODE: connectStores() per action + IOManager.SetParent() local-then-parent resolution

An action may re-declare a store name that already exists at payload scope. Go connects each action's stores separately and resolves `GetStore` local-first, so **the action's declaration shadows the payload's** — here changing both the backend (S3 to FS) and the root. An SDK that flattens the two scopes picks the wrong connection and reads the wrong bucket.

The shadowing is the entire value of action-scoped stores; it is invisible in the docs and easy to implement as a merge. Same-name-different-config must be legal, not a validation error, and must resolve action-first.
