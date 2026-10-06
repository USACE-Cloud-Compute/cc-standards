# basic-mapping

**Pins:** `STD-WIRE-020 / STD-WIRE-021`  ·  **Status:** PINNED
**Derived from:** DOC: cc-home/docs/06_compute-manifest.md field table and inputs sample

**The reference mapping, and the reason these fixtures exist.** Read left to right:

| Compute manifest | Payload |
|---|---|
| `inputs.payload_attributes` | `attributes` |
| `inputs.data_sources` | `inputs` |
| `inputs.environment` | *consumed by the provider, not present in the payload* |
| `inputs.parameters` | *consumed by the provider for `Ref::` in `command`* |
| `outputs` | `outputs` |
| `actions` | `actions` |
| `stores` | `stores` |
| `retry_attempts`, `job_timeout`, `resource_requirements`, `tags`, `dependencies`, `plugin_definition`, `manifest_name` | *not in the payload at all* |

Two consequences that are easy to get wrong:

1. **The rename `payload_attributes` -> `attributes` is the exact spot where the
   `payloadAttributes` doc error enters.** An author who reads the compute-manifest docs
   writes `payload_attributes` (correct); an SDK author who reads the Confluence payload
   example reads `payloadAttributes` (wrong). Only the *serialized payload* settles it, and
   the Go tag says `attributes`. See `payload/attributes-key-divergence`.
2. **`inputs.environment` must NOT appear in the payload.** It is injected into the
   container by the provider. An SDK that copies it into the payload leaks values that were
   meant for the process environment into a stored, readable document.
