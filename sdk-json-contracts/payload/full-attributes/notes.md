# full-attributes

**Pins:** `STD-WIRE-021 / STD-SDK-001`  ·  **Status:** PINNED
**Derived from:** DOC: cc-home 06 inputs.payload_attributes; CODE: attribute type coercion in payload-attributes.go

Payload attributes are `map[string]any` in Go. Every JSON scalar type must survive a round trip **with its type preserved**. The coercion helpers (`GetInt`, `GetBoolean`, `GetFloatSlice`) cast on read, which means an SDK that stringifies on parse will silently break `GetBoolean` for `false` (Go casts via string: `strings.ToLower(s) == "true"`). Note `"01"` is a string, not the integer 1: plan numbers with leading zeros lose them if any SDK round-trips through a numeric type.
