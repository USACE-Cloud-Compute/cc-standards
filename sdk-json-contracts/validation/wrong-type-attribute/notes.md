# wrong-type-attribute

**Pins:** `STD-WIRE-021 / STD-TST-006`  ·  **Status:** OPEN
**Derived from:** CODE: GetAttribute[T] uses cast.ToInt64E / cast.ToStringE+lower=='true'

Go's attribute accessors **coerce**. `GetInt` on `"not-a-number"` returns an error, but `GetIntOrDefault` logs and returns the default, and `GetBoolean` routes through `cast.ToStringE` then `lower == "true"` — so `"false"`, `false`, `0` and `"0"` all have to be checked individually against that expression. Note `GetOrFail` calls `log.Fatalf`, which **terminates the process** and is precisely what STD-SDK-009 forbids in library code: a plugin cannot map that onto the exit-code taxonomy.

Whether the schema should be strict (attributes are strings only, per the Go-generated schema) or permissive (any) is open. It is currently typed `additionalProperties: true` here because that matches reality; the Go-generated JSON schema says string, which does not.
