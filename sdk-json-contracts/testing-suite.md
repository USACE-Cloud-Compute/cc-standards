# Cross-SDK testing suite

**Implements:** `standards.md` STD-SDK-001, STD-SDK-002, STD-SDK-005, STD-WIRE-010.

This directory is the only mechanism that keeps four independently released SDKs honest about
one JSON format.

## What testing establishes

An SDK is conformant at contract version `N` when it produces the expected result for **every**
fixture here. Not "passes most". Not "passes its own tests". A language **MUST NOT** tag a release until this suite runs green in that language's CI
(`standards.md` STD-SDK-001).

## Layout

```
sdk-json-contracts/
├── testing-suite.md          # this file: the runner protocol
├── schema/                       # 7 documents; the schema IS the contract (STD-WIRE-020)
│   ├── payload.schema.json           actions[] -> action.schema.json
│   ├── action.schema.json            stores[]/inputs[]/outputs[] -> the two below
│   ├── data-store.schema.json
│   ├── data-source.schema.json
│   ├── plugin-manifest.schema.json   registration document
│   ├── compute-manifest.schema.json  authored document
│   └── compute-file.schema.json      CLI run descriptor
├── payload/          (11)  golden payload -> expected parsed model
│   minimal · full-attributes · multi-store · multi-action-ordered
│   unknown-keys                  STD-WIRE-003 -- currently fails in every SDK
│   attributes-key-divergence     attributes vs payloadAttributes   [F-1]
│   action-params-key             attributes vs params              [F-2]
│   action-name-vs-type           dispatch field                    [F-3]
│   action-store-shadowing        action scope shadows payload scope
│   duplicate-datasource-name     first-match vs error              [open]
│   outputs-not-readable          outputs are not readable
├── substitution/     (12)  STD-WIRE-010..015
│   attr-resolves · env-resolves · nested-attr-in-path · partial-token-literal
│   name-substitution · data-paths-substitution
│   missing-attr-is-fatal · missing-env-is-fatal
│   empty-value-is-fatal          "" vs absent -- needs LookupEnv    [STD-WIRE-015]
│   unclosed-token · recursive-token · self-referential-attr
│   store-params-not-substituted  Go and Python disagree             [F-5]
│   scope-payload-vs-action       the substituteMapVariables(..., bool) asymmetry
│   braces-in-legit-path          {([^{}]*)} matches ANY braced token [F-4]
├── validation/       (5)   malformed input -> error class + JSON pointer
│   missing-name · bad-store-type · wrong-type-attribute
│   unknown-top-level-key · empty-collections-valid
├── store/            (8)   backend behaviour parity (STD-STR-002)
│   listing-order · prefix-vs-directory · missing-object · roundtrip-bytes
│   multipart-datasource · write-then-list-consistency
│   root-prefix-join
│   root-escape-rejected        STD-SEC-020 -- NOT enforced today     [F-14]
└── transform/        (5)   compute manifest -> payload (the mapping is contract)
    basic-mapping · naming-collisions · dependency-by-filename
    generator-expansion · absent-inputs-block
```

36 fixtures. Every `expected.json` carries `_meta.status` of `PINNED` (settled, and an SDK
that disagrees has a bug) or `OPEN` (unresolved; a FAIL is a **finding**, not a defect). 25
PINNED / 11 OPEN. That ratio is deliberate: a contract pack that presents 36 settled answers
when 11 questions are live is a pack that will be discarded the first time it contradicts
production behavior.

## Fixture format

Each fixture is a directory:

| File | Purpose |
| --- | --- |
| `input.json` | The payload or manifest as the platform delivers it |
| `env.json` | Environment variables present at load time |
| `expected.json` | Canonical parsed model, or `{"error": "<code>", "pointer": "/..."}` |
| `notes.md` | Why this case exists, and which clause it pins |

`expected.json` is compared **structurally after normalization**, never as raw bytes.
Normalization: object keys sorted, numeric values compared with the fixture's declared
tolerance, field omission treated as null. Byte comparison would fail four correct SDKs
against each other over serialization trivia and would teach contributors to distrust the
suite — which is worse than having no suite.

## When to add a fixture

Add one whenever:

1. A new payload/manifest field or environment variable is introduced.
2. A bug is fixed in any SDK's parsing, substitution, validation, or store behaviour — add the
   fixture **before** porting the fix to the other three, so the suite tells you which are
   affected. This is the practical form of STD-SDK-001.
3. Two SDKs are discovered to disagree. Whichever behaviour is chosen becomes the fixture; the
   other side becomes a bug.
4. An ADR changes semantics. The ADR names the fixtures it adds or changes.

## Versioning

`contract_version` is an integer, bumped **only by ADR**. Every SDK release records the
contract version it satisfies (STD-SDK-002). CI in `cc-standards` runs every SDK adapter
nightly against the current `contract_version`, and the compatibility matrix in
`standards.md` Appendix B is generated from those runs — never hand-edited.

## Answers already established by reading the code

Several questions below were posed as "the first run will tell us". Reading `cc-go-sdk` and
`cc-py-sdk` answered them, and three of the answers reversed the draft:

| Question | Answer | Evidence |
| --- | --- | --- |
| Is the attributes key `attributes` or `payloadAttributes`? | **`attributes`.** `payloadAttributes` exists only in documentation | Go ``json:"attributes"``; Python `Payload.attributes` |
| Are action parameters under `params` or `attributes`? | **`attributes`.** No implementation parses `params` | Go `Action` embeds `IOManager`; Python `Action.attributes` |
| Do the SDKs dispatch on `name` or `type`? | **`name`, both.** The Python README example is misleading | Go `RunActions()`; Python `get_action_runner(action.name)` |
| Is substitution scoped to `{ENV::}`/`{ATTR::}`? | **No.** Both use the identical literal `{([^{}]*)}` -- any braced token | `substitutionRegexPattern` / `substitutionPattern` |

Still genuinely open, and the reason those fixtures stay `OPEN`:

| Question | Fixture | Why it is open |
| --- | --- | --- |
| `""` vs absent on substitution | `empty-value-is-fatal` | `os.Getenv` cannot distinguish them; needs `LookupEnv` |
| Recursive expansion? | `recursive-token`, `self-referential-attr` | Security-relevant; needs a decision, not an observation |
| Store `params` substitutable? | `store-params-not-substituted` | **Python says yes, Go says no** -- divergence, not ambiguity |
| Unknown-key preservation | `unknown-keys` | **No SDK does it.** A MUST nobody meets needs an ADR, not a fixture |
| Duplicate DataSource names | `duplicate-datasource-name` | Go silently picks the first; recommend erroring |
| FS backend parity | `store/*` | Python is **S3-only**; it cannot run these fixtures at all |

Full evidence trail, including three suspected bugs (F-8 `id` dropped on Python
re-serialize, F-10 `action.inputs()` shadowed by a dataclass field, F-11 `get_store`
returning an uncalled method), is in **`FINDINGS.md`**.

| Question | Why it matters |
| --- | --- |
| Does `{ATTR::x}` where `x` is present-but-empty substitute `""` or fail? | `""` yields a plausible wrong path — the worst failure mode in §5.2 |
| Are substitution tokens resolved recursively? | A value containing `{ENV::...}` could reach a variable the plugin never declared |
| Which payload fields are substitutable? | "file path, data path, or datasource name" per docs; `store_name` and `params` are unaddressed |
| Is an unknown key preserved or dropped on re-serialize? | **Answered: dropped, in every SDK** (F-7) |
| How are duplicate DataSource names resolved? | **Answered: first match wins, silently**; recommend erroring at load |

Document each answer in `notes.md`, then make it a MUST in `standards.md` §5 with a fixture.
