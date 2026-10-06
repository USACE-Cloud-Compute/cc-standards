# empty-value-is-fatal

**Pins:** `STD-WIRE-010 / STD-WIRE-011`  ·  **Status:** OPEN
**Derived from:** CODE: Go `os.Getenv` returns "" for both unset and set-empty; the regex cannot distinguish them without a lookup against the source map

**The dangerous case.** Two sub-questions the code does not obviously settle:

1. Attribute `empty` is present with value `""`. Substitute to `""` (giving `/out.hdf`, a *valid-looking* path) or fail?
2. Env var `EMPTY_VAR` is exported as empty. In Go `os.Getenv` returns `""` for set-to-empty and for unset — an SDK using only `os.Getenv` **cannot tell these apart at all**, so 'missing is fatal' cannot be implemented for env vars without `os.LookupEnv`.

Substituting empty produces `models//out.hdf`, which frequently resolves to *something*, so the plugin computes on the wrong object and returns a plausible wrong number. Absent-vs-empty is precisely the distinction that decides between a loud startup failure and a silently wrong FFRD result at scale. Recommend: fatal for both, and require `LookupEnv`-equivalent semantics in every SDK. Needs an ADR, because making it fatal may break payloads that today rely on empty substitution.
