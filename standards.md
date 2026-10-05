# Cloud Compute Development Standards

**Version:** 0.1 (proposed)

**Status:** Draft for FFRD Software Development Strategy

**Precedence:** Layer 1. See `README.md` §3. Overrides `styles.md` and `CONTRIBUTING.md` overridden only by Layer 0 federal/DoD/USACE policy.

**Terminology:** Plugin, Payload, DataSource, DataStore, Action, Manifest, Event, DAG,
Compute Provider are defined in
[`cc-home/docs/08_glossary.md`](https://github.com/USACE-Cloud-Compute/cc-home/blob/main/docs/08_glossary.md).
This document uses them with exactly those meanings and defines no new terms for them.

Keywords **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, **MAY** carry their RFC 2119
meanings. Each MUST carries a stable ID (`STD-<AREA>-<NNN>`) for citation in pull requests,
CI messages, and `CC-DEVIATIONS.md` entries.

**This document does not cover:** formatting, indentation, naming case, docstring/comment
syntax, lint configuration (`styles.md`); branch naming, pull request mechanics, review
routing (`CONTRIBUTING.md`).

---

## 1. Scope and repository taxonomy

### 1.1 Repo classes

| Class | Contains | Additional obligations |
| --- | --- | --- |
| **Platform** | Abstraction Layer, CLI, storage APIs (`cloudcompute`, `cloudcompute-cli`, `filesapi`) | Full §5 wire rules, §11 API compatability, two-person review |
| **SDK** | Language plugin SDKs (`cc-go-sdk`, `cc-java-sdk`, `cc-py-sdk`, `cc-dotnet-sdk`) | All of §5, §7, §8, §11 — including `contracts/` conformance |
| **Plugin**| Wraps an existing model executable (`hms-runner`, `consequences-runner`, `slam-plugin`, `storm-cloud-plugin`) | §5, §6, §9, §10 (native dependency rules), §12 |
| **Documentation** | `cc-home`, `ffrd-demo-directory` | §13 only |

### 1.2 Naming

- New repository names **MUST** be lowercase, hyphen-delimited, ASCII: `^[a-z][a-z0-9-]{1,38}$`.
- `cc-` prefix reserved for platform and SDK. Plugin/Adapter/Model wrappers repos **SHOULD** end in `-plugin` or `-runner`.
- **STD-REPO-001** — Existing names that violate these (e.g. `cc_py_sdk` underscore form in documentation) are **grandfathered**. Do not rename
  a repository to satisfy naming. Renames break registered container images, module paths,
  and PyPI/NuGet coordinates — a far larger cost than the inconsistency.
- **STD-REPO-002** — Every repository **MUST** declare its class in the first line of its
  README, so tooling and newcomers can classify without guessing.

---

## 2. Language and platform baseline

- **STD-LANG-001** — Each repository **MUST** pin its language/toolchain version in exactly
  one place per ecosystem: `go.mod` `go`, `pyproject.toml` `requires-python`,
  `build.gradle` toolchain, `*.csproj` `TargetFramework` — plus a `.tool-versions` or
  `mise.toml` for developer machines.
- **STD-LANG-002** — CI **MUST** test the minimum supported version, not only the latest.
  The `cc-py-sdk` guide historically advertised Python 3.5–3.8; a floor that no build tests
  is not a floor.
- **STD-LANG-003** — Supported version floors, current proposal: Go ≥ 1.22, Python ≥ 3.11,
  Java ≥ 17 (LTS), .NET ≥ net8.0 (LTS). Every language version **MUST** be under upstream
  security support. Dropping a floor is a minor SDK release; adding one requires an ADR.
- **STD-LANG-004** — Build **MUST** be reproducible and non-interactive from a clean clone:
  `make ci` or equivalent one command, no undocumented manual step. If the documented command
  and the CI pipeline disagree, the pipeline is right and the docs are a defect.
- **STD-LANG-005** — Build for dev and tests **MUST NOT** require a developer-specific absolute path,
  host-only tool, or the developer's cloud credentials.
- **STD-LANG-006** — Plugins **MUST** target `linux/amd64/arm64` unless an ADR records another
  target, since the Compute Provider provisions that architecture.

---

## 3. Licensing, attribution, provenance

- **STD-LIC-001** — Platform and SDK code **MUST** be MIT, matching `cloudcompute`.
- **STD-LIC-002** — Plugin license **MUST** be declared in the repository (`LICENSE` file and
  the ecosystem metadata field: `pyproject.toml` `license`, Gradle `license`, `*.csproj`
  `PackageLicenseExpression`, and the container `org.opencontainers.image.licenses` label).
  Undeclared is a merge blocker — several repositories in this org currently report
  `license: null` on GitHub, which reads as all-rights-reserved to any downstream consumer.
- **STD-LIC-003** — **OPEN DECISION D-4.** Proposed: MIT, Apache-2.0, BSD-2/3-Clause, ISC
  pre-approved. GPL/AGPL, proprietary, "USACE internal use only", or export-controlled
  components require a recorded license review before merge. `cloudcompute` already warns that
  licensing terms may prevent deployment; that warning needs to become a gate.
- **STD-LIC-004** — Copyleft **MUST NOT** enter a platform or SDK dependency tree. Plugin
  dependency trees may include LGPL only with recorded review. **MUST NOT** link AGPL code
  into anything.
- **STD-LIC-005** — Third-party notices **MUST** be complete for redistributed binaries.
  Vendored or embedded code carries its original license and a `NOTICE` entry.
- **STD-LIC-006** — Federal-employee-authored code is not copyrightable; contractor-authored
  code is, and the license is granted by the contract. **MUST NOT** add a copyright line
  implying USACE ownership of contractor code. `AUTHORS`/`NOTICE` files record provenance
  factually. (`cc-py-sdk` carries a cookiecutter-derived `AUTHORS.rst`; verify before
  inheriting it further.)
- **STD-LIC-007** — Every release artifact **MUST** ship a CycloneDX or SPDX SBOM. Required
  for software assurance attestation under OMB M-22-18 / M-23-16.

---

## 4. Data classification and repository contents

- **STD-DATA-001** — CC repositories and their fork network **MUST NOT** contain PII, PHI, or
  CUI. This restates the explicit rule in `cloudcompute` as an enforced gate.
- **STD-DATA-002** — Because these repositories are public with forking enabled, the fork
  network is not controllable. A secret or classified fixture pushed to a fork cannot be
  reliably removed org-wide. **MUST NOT** accept test fixtures derived from real
  post-event survey data, real damages, real infrastructure ownership records, or
  personally identifying gauge-station contact data.
- **STD-DATA-003** — Test fixtures **MUST** be synthetic, or public-domain/baseline data with
  documented provenance and redistribution rights. Every fixture directory **MUST** carry a
  `PROVENANCE.md` naming its source.
- **STD-DATA-004** — Secrets **MUST NOT** appear in source, tests, fixtures, manifests,
  Dockerfiles, CI logs, or commit messages — including in example files. Use unambiguous
  placeholders (`CC_AWS_SECRET_ACCESS_KEY=REPLACE_ME`). `secretsmanager:<name>::` references in
  the manifest `credentials` block are the sanctioned pattern; literals never are.
- **STD-DATA-005** — Internal hostnames, tenant IDs, account numbers, and network topology
  **SHOULD NOT** appear in public repos; they are reconnaissance value for free.
- **STD-DATA-006** — Plugins and SDKs **MUST** be locale-neutral in storage and interchange:
  UTF-8, `.` decimal separator, ISO 8601 UTC (see §10).
- **STD-DATA-007** — Log output **MUST NOT** contain credentials, signed URLs, or bulk
  subject-level data. Log a DataSource name, not its contents.

---

## 5. The wire contract — payload, manifest, environment

This section governs the interfaces that make four independently released SDKs and every
plugin behave as one system. It is the most consequential section in this document, and the
one most likely to be broken accidentally.

### 5.1 Why it is append-only

Plugin container images are **registered** with a Compute Provider and referenced by
`image_and_tag` in manifests that may already exist in customer projects and long-lived FFRD
DAGs. Consequences:

- **STD-WIRE-001** — The meaning of an existing JSON key in a plugin manifest, compute
  manifest, compute file, or payload **MUST NOT** change.
- **STD-WIRE-002** — A new field **MUST** be optional with a defined default and **MUST NOT**
  be required by any SDK that reports a compatible minor version.
- **STD-WIRE-003** — Unknown keys **MUST** be preserved on round-trip (parse → modify →
  serialize) and **MUST NOT** be silently dropped. Strict-mode deserialization that rejects
  or discards unknown fields is prohibited in SDKs. Without this, a plugin built against an
  older SDK corrupts a manifest written by a newer CLI — and it will do so silently, in
  production, at scale.
- **STD-WIRE-004** — A key **MUST NOT** be removed until no supported SDK or CLI can emit it,
  and only after the §11 deprecation window.
- **STD-WIRE-005** — **Key casing is frozen and mixed.** The existing contract uses
  `payloadAttributes` alongside `store_name`, `store_type`, `image_and_tag`,
  `compute_environment`, `retry_attempts`, `execution_timeout`, `linux_parameters`. This is
  ugly. It is also load-bearing. **MUST NOT** "normalize" casing during cleanup, refactoring,
  or a linter's opinion about JSON keys — it is a breaking change under STD-WIRE-001. New
  keys **MUST** be camelCase; legacy snake_case keys are recorded as frozen exceptions in
  Appendix C and are permanent unless superseded by an ADR.

### 5.2 Substitution semantics

Payloads support `{ENV::VAR}` and `{ATTR::name}` substitution in file paths, data paths, and
DataSource names; the documented behavior is fatal failure when a referenced variable or
attribute is absent.

- **STD-WIRE-010** — All SDKs **MUST** implement identical substitution semantics, verified
  by `contracts/substitution/`, including: which fields are substitutable, recursion rules,
  behavior on an empty vs missing value, and behavior on a partial match inside a longer
  string.
- **STD-WIRE-011** — Substitution failure **MUST** be fatal, non-zero exiting, with a message
  naming the missing variable or attribute and the field containing it. Silent empty-string
  substitution is prohibited: it produces a plausible-looking wrong path, and the plugin then
  computes on the wrong data.
- **STD-WIRE-012** — Substitution **MUST NOT** be applied to values that will be interpreted
  as shell, SQL, or filesystem paths without separate validation (§12.3).

### 5.3 Schema and validation

- **STD-WIRE-020** — Every manifest and payload structure **MUST** have a versioned JSON
  Schema in `contracts/`. The schema is the contract; README prose describing fields is
  documentation of it. Where they disagree, the schema wins and the prose is a defect.
- **STD-WIRE-021** — All SDKs and the CLI **MUST** validate against the schema and report
  field-level errors with a JSON pointer path.
- **STD-WIRE-022** — Every manifest and payload file in a repository **MUST** pass schema
  validation in CI (`manifest-validate`).
- **STD-WIRE-023** — The `command` array in a plugin manifest **SHOULD** be explicit rather
  than inherited from Docker `CMD`/`ENTRYPOINT`, per `cc-home/docs/05_plugin-manifest.md`, so
  registered behavior is inspectable and troubleshooting does not require rebuilding an
  image to learn what ran.

### 5.4 Environment variable namespace

Observed practice mixes namespaces: `CC_STORE_TYPE`, `CC_AWS_S3_BUCKET`, `CC_LOGGING_LEVEL`,
`CC_EVENT_NUMBER` alongside bare `AWS_REGION`, `AWS_S3_BUCKET`, `AWS_S3_ENDPOINT`,
`AWS_HTTPS`, `AWS_VIRTUAL_HOSTING`, `FSB_ROOT_PATH`.

- **STD-ENV-001** — `CC_` prefix is reserved for CC-defined variables. `AWS_` is reserved for
  variables consumed natively by AWS SDKs. **MUST NOT** overload either prefix with CC-specific
  meaning, and **MUST NOT** introduce a new prefix.
- **STD-ENV-002** — `FSB_ROOT_PATH` is grandfathered; the target is `CC_FSB_ROOT_PATH` read
  with `FSB_ROOT_PATH` as fallback, per the §11 deprecation path.
- **STD-ENV-003** — Every variable read by a plugin or SDK **MUST** appear in Appendix A
  (the environment registry) with name, meaning, type, default, whether it is a credential,
  and the SDK versions supporting it. Undocumented reads are a merge blocker.
- **STD-ENV-004** — Code **MUST NOT** read an environment variable not present in the
  registry, and **MUST** fail with a named error when a required one is absent (§5.2).
- **STD-ENV-005** — Configuration **MUST** come from environment and payload only. **MUST
  NOT** read a developer-machine config file (`~/.aws/credentials`, a local `.ini`) in plugin
  runtime code: it does not exist in the container, and a plugin that works locally and fails
  in the cloud costs a full scheduling round-trip to discover.

---

## 6. Plugin runtime contract

Any container scheduled by CC, in any language, SDK-based or not.

### 6.1 Isolation

- **STD-PLG-001** — A plugin **MUST** be fully containerized and self-contained at run time.
- **STD-PLG-002** — A plugin **MUST NOT** communicate with another plugin. Coordination is
  exclusively through DataSources in DataStores, ordered by the DAG. No sockets between
  siblings, no shared scratch directory, no file-based handshake.
- **STD-PLG-003** — A plugin **MUST NOT** spawn or submit other CC compute. Scaling is the
  platform's responsibility.
- **STD-PLG-004** — A plugin **MUST NOT** write outside its designated working directory, and
  **MUST NOT** assume any local path persists between invocations. A local filesystem is
  scratch, not storage: outputs reach the outside world only via DataSources.
- **STD-PLG-005** — A plugin **MUST** read only DataSources granted by its payload and
  **MUST** write only to its declared outputs. **MUST NOT** enumerate buckets or prefixes
  beyond its grant.
- **STD-PLG-006** — A plugin **MUST** tolerate an empty, absent, or partially populated
  optional input and report which it received.

### 6.2 Exit status and signaling

- **STD-PLG-010** — Exit `0` **MUST** mean success. **MUST NOT** exit `0` on a partial or
  degraded run. Retry policy and DAG downstreams are driven by exit code; a lying exit code
  silently corrupts a massively parallel run and is nearly impossible to audit afterward.
- **STD-PLG-011** — Failure **MUST** use a non-zero code from the registered set:

  | Code | Class | Retryable | Meaning |
  | --- | --- | --- | --- |
  | `0` | success | — | All declared outputs written and verified |
  | `1` | internal error | no | Unexpected exception/panic |
  | `2` | usage/config error | no | Malformed payload, missing attribute, bad manifest |
  | `3` | input error | no | Declared input absent, unreadable, or failed validation |
  | `4` | transient dependency error | yes | Store/network timeout, throttle |
  | `5` | resource exhaustion | yes, after resize | OOM, disk full, timeout inside plugin |
  | `6` | numerical failure | no | Solver diverged, non-convergence, invalid result domain |
  | `7` | license/entitlement | no | Wrapped model refused to run |
  | `137`/`143` | platform kill | per policy | SIGKILL/SIGTERM; do not invent these |

- **STD-PLG-012** — A plugin **MUST** handle `SIGTERM` within the platform grace period: flush
  buffers, close stores, emit a terminal log record, then exit non-zero. **MUST NOT** trap
  `SIGTERM` to keep running.
- **STD-PLG-013** — A plugin **SHOULD** declare `execution_timeout` in its manifest. Relying
  on platform defaults converts a hung run into a billing incident.
- **STD-PLG-014** — A retryable failure **MUST** be idempotent: re-running must not duplicate
  or corrupt a prior partial write. Write outputs to a temporary key/path and move on
  completion, or write to an event-scoped prefix that is safe to overwrite.

### 6.3 Logging

- **STD-PLG-020** — Human-readable logs to `STDOUT`; errors and diagnostics to `STDERR`.
  **MUST NOT** write structured payload data, model binaries, or results to `STDOUT` — that
  stream is a captured log, not an output channel.
- **STD-PLG-021** — Log level **MUST** honor `CC_LOGGING_LEVEL`, defaulting to `INFO`.
- **STD-PLG-022** — Every log line **SHOULD** include the event and plugin identity, so lines
  from thousands of parallel runs remain attributable. A bare `INFO: done` is worthless in a
  10,000-event batch.
- **STD-PLG-023** — **MUST NOT** emit credentials, tokens, or signed URLs — including at debug
  level, where they are routinely captured by the log sink.
- **STD-PLG-024** — On failure a plugin **MUST** emit a single terminal `error` record naming
  the failure class, the affected DataSource, and the exit code, before exiting. Status
  propagation to MQTT is reserved for the SDK/platform layer; a plugin **MUST NOT** open its
  own MQTT connection unless an ADR says otherwise.

### 6.4 Resources and security posture

- **STD-PLG-030** — `compute_environment.vcpu` and `.memory` **MUST** be measured, not
  guessed: recorded from a representative run with headroom, and documented in the plugin
  README. Under-declared memory is OOM-kill roulette across a fleet; over-declared is the
  dominant cost driver in massively parallel FFRD runs.
- **STD-PLG-031** — `privileged` **MUST** default to `false`. Setting it `true` requires an
  ADR plus a named device in `linux_parameters.devices` — a privileged container is nearly
  host-equivalent and forfeits the isolation STD-PLG-001 exists to provide.
- **STD-PLG-032** — Plugin images **MUST** run as a non-root user unless the wrapped
  executable genuinely requires root, recorded in the Dockerfile as a comment.
- **STD-PLG-033** — `Dockerfile` **MUST** be multi-stage (builder + minimal runtime), per the
  existing pattern in `cc-home`'s Dockerfile guidance, and pin base images by tag with a
  documented upgrade path.
- **STD-PLG-034** — Runtime image **SHOULD** be minimal; **MUST NOT** include compilers,
  package-manager caches, `.git` metadata, or build secrets.
- **STD-PLG-035** — Image **MUST** carry OCI labels: `org.opencontainers.image.title`,
  `.source` (repo URL), `.revision` (commit SHA), `.created`, `.licenses`. An image running in
  production with no resolvable source commit is unpatchable and unauditable.
- **STD-PLG-036** — Network egress **MUST** be declared in the README and limited to what is
  declared. A plugin phoning home is both a security finding and a scaling cliff.

---

## 7. SDK obligations and cross-language parity

- **STD-SDK-001** — Every SDK **MUST** pass the `contracts/` conformance suite: byte-identical
  parse results for the golden payloads/manifests, identical substitution outcomes including
  failure cases, identical validation error coverage. **A language SDK that has not run the
  suite MUST NOT claim conformance in its README.**
- **STD-SDK-002** — Parity is a release gate. An SDK release **MUST** record the
  `cc-standards` contract version it satisfies.
- **STD-SDK-003** — Divergence is a defect, not a feature, unless an ADR records a
  language-idiomatic equivalence. `get_payload()` in Python and `GetPayload()` in C# are
  equivalent; a Python method that tolerates a missing attribute where C# throws is not.
- **SDK-004** — Concept coverage: `PluginManager`, `Payload`, `DataSource`, `DataStore`,
  `IOManager`, `Action`, `PayloadAttributes` **MUST** all be reachable in every SDK. Where a
  concept is genuinely absent (documented as in development, e.g. the TileDB store in
  `cc-go-sdk`), the README **MUST** say so plainly rather than presenting a uniform API.
- **STD-SDK-005** — SDKs **MUST** expose the same storage backend set, and every backend
  **MUST** be exercised by the suite. A backend present in one SDK only is a portability trap
  for plugin authors.
- **STD-SDK-006** — SDKs **MUST NOT** require a network connection or cloud credentials to
  run their own unit tests. Use the `FS`/`FSB` backend or an in-memory/S3-mock store.
- **STD-SDK-007** — Public API surface **MUST** be explicitly bounded: an exported-symbol list
  or `__all__`/module-info/`internal/` package/`public` filter. Undeclared surface becomes
  depended on within one release and is then unremovable.
- **STD-SDK-008** — SDKs **MUST NOT** print to `STDOUT`/`STDERR` directly; **MUST** use the
  logging facility (§6.3) so plugin authors control level and sink.
- **STD-SDK-009** — SDKs **MUST** return/propagate errors rather than terminating the process,
  except for unrecoverable startup contract failures. **MUST NOT** call `os.Exit`, `System.exit`,
  `Environment.Exit`, or `panic` inside library code paths — the host plugin loses the ability
  to map the failure onto §6.2's exit codes.
- **STD-SDK-010** — SDKs **MUST** document the minimum plugin-visible behavior when running
  outside a CC container, so plugins can be developed against local Docker compute.

---

## 8. Storage access

- **STD-STR-001** — All data access **MUST** go through the SDK store abstraction. **MUST NOT**
  instantiate a raw S3 client, open a mounted path, or construct a URL from bucket + key in
  plugin code. Every direct bypass is a hidden assumption that the backend stays `S3`.
- **STD-STR-002** — Plugin code **MUST** behave identically across `FS`/`FSB` and `S3`. CI
  **MUST** run the storage-touching tests against both. Real divergence: path separators, key
  prefix vs directory semantics, listing behavior, "directory" existence, case sensitivity,
  eventual consistency after a write, streaming vs seekable reads.
- **STD-STR-003** — **MUST NOT** assume a listing is sorted; **MUST NOT** assume an object
  appears in a listing immediately after write. Sort explicitly by key.
- **STD-STR-004** — Streaming **MUST** be used for objects above a declared size threshold;
  **MUST NOT** buffer an unbounded remote object in memory. A model library or raster is not a
  byte array.
- **STD-STR-005** — Keys/paths **MUST** be built by joining declared components with an
  explicit separator function, never by `+`-concatenating unvalidated strings (§12.3).
- **STD-STR-006** — Multi-part file DataSources (e.g. `.prj`/`.shp`/`.dbf`) **MUST** be handled
  as a unit: all-or-nothing staging, consistent ordering, all components written before the
  output DataSource is published. A `.shp` without its `.dbf` is a corrupt output that may not
  fail until a downstream plugin reads it.
- **STD-STR-007** — Store selection is by configuration (`CC_STORE_TYPE`), never hardcoded.

---

## 9. Dependencies

- **STD-DEP-001** — All direct dependencies **MUST** be pinned to an exact version and recorded
  in a committed lockfile: `go.sum`, `uv.lock`/`requirements.txt` hashes, `gradle.lockfile`,
  `packages.lock.json`. Floating ranges (`latest`, `*`, `>=`, bare `main`) are prohibited in
  anything that produces a release artifact.
- **STD-DEP-002** — No dependency may resolve to an unreviewed transitive graph: `go mod
  tidy` in CI, `pip-audit`/lock audit, Gradle dependency locking verification, NuGet lock
  verification.
- **STD-DEP-003** — Adding a dependency **MUST** state in the pull request: what it does, why
  existing dependencies cannot do it, its license, its maintenance status, and its transitive
  cost.
- **STD-DEP-004** — New dependencies in platform or SDK code require an ADR when they add a
  system-level dependency, a network service, a build-time code generator, or anything under a
  non-approved license.
- **STD-DEP-005** — Dependency pull requests are separate from feature pull requests, always.
- **STD-DEP-006** — **Native and binary dependencies** (GDAL/PROJ, Fortran runtimes, `libgfortran`,
  model executables, `HEC-HMS`/`RAS` distributions, TileDB C libraries) **MUST** be pinned by
  exact version **and** checksum, vendored from an approved internal mirror, or fetched from a
  declared URL with a verified digest. **MUST NOT** rely on `apt install <pkg>` resolving to
  "whatever is current": the `hms-runner`-style Dockerfiles in this program download tarballs
  over HTTP, and an unpinned mirror change silently alters numerical output.
- **STD-DEP-007** — `GDAL_DATA` / `PROJ_LIB` **MUST** point at data shipped in the same image
  as the library that reads it. A version-skewed PROJ database produces wrong coordinates,
  not an error — the worst possible failure mode for FFRD output.
- **STD-DEP-008** — Every dependency **MUST** have a named owner and a documented upgrade path.
  An unmaintained dependency in a plugin used by a production DAG is a program risk, and
  `dependency-audit` reports it whether or not anyone opens a ticket.

---

## 10. Testing

### 10.1 Required layers

| Layer | Scope | Runs on |
| --- | --- | --- |
| **Unit** | Functions/methods, no network, no cloud | Every PR |
| **Contract** | Schema validation of committed manifests/payloads | Every PR |
| **Cross-SDK conformance** | `contracts/` fixtures | Every SDK PR + nightly |
| **Integration** | Plugin against local `FS`/`FSB` store | Every PR |
| **End-to-end** | Plugin in its real image via local Docker compute, full mini-DAG | Release, and nightly |
| **Golden/regression** | Numeric output vs stored baselines | Every PR for numeric code |

- **STD-TST-001** — Every MUST in §5–§9 **SHOULD** have a test; every bug fix **MUST** ship a
  test that fails without the fix. Confirm by reverting the fix and watching it fail — a test
  that never failed is not evidence of anything.
- **STD-TST-002** — Tests **MUST** be hermetic: no ordering dependence, no shared mutable
  state, no wall-clock or timezone assumptions, no ambient credentials. **MUST** pass under
  random-order execution and in parallel.
- **STD-TST-003** — Coverage floors: SDK ≥ 80% lines on changed code; plugin ≥ 70%. Coverage
  is a floor, not a target — a 100%-covered store abstraction that never asserts on key
  construction has told you nothing.
- **STD-TST-004** — Numeric plugins **MUST** have golden tests with an explicitly justified
  tolerance, and **MUST** state whether the tolerance is absolute, relative, or
  units-per-item. `assertAlmostEqual(x, y)` without a stated tolerance is an unfalsifiable test.
- **STD-TST-005** — Golden baselines **MUST** record the platform, dependency versions, and
  producing command. A baseline that cannot be regenerated cannot be maintained.
- **STD-TST-006** — Negative paths **MUST** be tested: missing input, absent attribute,
  malformed payload, store unavailable, empty input set, oversized input, mid-write failure.
  In a system where the failure mode is a bad answer at 10,000-event scale, negative tests
  carry more information than positive ones.

### 10.2 Determinism and reproducibility

A program that must reproduce flood risk results cannot tolerate nondeterminism.

- **STD-DET-001** — Any randomness **MUST** use an explicitly seeded generator passed in
  configuration or payload attributes. **MUST NOT** use an unseeded global RNG, and **MUST
  NOT** seed from the current time.
- **STD-DET-002** — Aggregation order over parallel or partitioned results **MUST** be
  deterministic. `sum()` over a hash-ordered map, a set, or a thread-completion order is
  prohibited; reduce in a stable, explicitly sorted key order. Floating-point addition is not
  associative, so unordered reduction is nondeterministic *bit for bit*.
- **STD-DET-003** — Ties **MUST** be broken deterministically by a documented, stable key.
  "Max damage per block" with two equal rows is a real case, and which one wins must not depend
  on file read order. When a tie represents genuinely split credit, split it explicitly and
  document the rule.
- **STD-DET-004** — Monetary and other exactly-represented quantities **MUST NOT** use binary
  floating point. Use integer minor units or a decimal type; convert at the boundary, never
  `int(float(x) * 100)`, which is lossy. (`int(float("574211.69") * 100)` is `57421168`.)
- **STD-DET-005** — Numeric code **MUST** be locale-independent: explicit decimal separator,
  explicit encoding on every file open, no reliance on the container's `LANG`.
- **STD-DET-006** — Time handling: all persisted timestamps **MUST** be UTC, ISO 8601, with an
  explicit offset. **MUST NOT** encode wall-clock time into output keys or paths. **MUST NOT**
  do calendar arithmetic in local time — DST transitions silently change durations. Where a
  plugin legitimately needs "run time", it **MUST** be an input attribute, not
  `now()`, or the run is not reproducible.
- **STD-DET-007** — Every run **MUST** be attributable to: git commit SHA, dependency
  lockfile digest, container image digest, and payload. Emit all four at startup.
- **STD-DET-008** — Output **MUST** be byte-stable across runs given identical inputs and
  versions: no embedded timestamps, no map-iteration-order artifacts, no nondeterministic
  serialization. Byte-stable output is what makes a diff a usable review tool on a generated
  artifact.

---

## 11. Versioning, releases, deprecation, compatibility

- **STD-VER-001** — SemVer 2.0.0. Pre-release identifiers permitted (`-beta.1`, `-rc.1`), used
  as they already are in this ecosystem (`4.11-beta.16`).
- **STD-VER-002** — **BREAKING CHANGE** = major, for SDK/platform. Because of §5.1, a change
  that is technically breaking but provably cannot affect any supported consumer (verified by
  searching usage, not by intuition) **MAY** be minor with the justification recorded.
- **STD-VER-003** — Every tagged release **MUST** have: a GitHub Release with human-readable
  notes, a `CHANGELOG.md` entry, and an artifact in the declared registry.
- **STD-VER-004** — Release tags are **immutable**. Never reuse or move a tag that has been
  pushed. `cc-dotnet-sdk` already carries the warning "Don't re-use release tags"; make it a
  rule, because a moved tag silently invalidates pinned plugin builds and the §10.2 audit trail.
- **STD-VER-005** — Releases **MUST** be cut from a protected branch by CI, never from a
  developer workstation.
- **STD-VER-006** — Each ecosystem's publish path is documented in `CONTRIBUTING.md` §9 with
  the exact commands. Current paths: PyPI (`cc-python-sdk`), GitHub Packages
  (`nuget.pkg.github.com/USACE/` for `Usace.CC.Plugin`), Gradle/Maven repository, Go module
  proxy via tags.
- **STD-VER-007** — **OPEN DECISION D-3** — Support policy. Proposed: latest minor of each SDK
  plus one prior receive fixes; security fixes 12 months beyond that; the compatibility matrix
  (Appendix B) is generated by CI from the matrix actually tested, not hand-maintained.
- **STD-VER-008** — Deprecation window: any public API, manifest field, or environment variable
  **MUST** warn for at least one minor release **and** 6 months before removal. Warnings go to
  the log at `WARN`, name the replacement, and cite the version after which the old form fails.
- **STD-VER-009** — An SDK **MUST** state the range of payload/manifest contract versions it
  accepts, and **MUST** reject an unsupported major contract version at load time with a clear
  message rather than partially parsing it.
- **STD-VER-010** — Where SDKs must be upgraded in lockstep (a shared contract change),
  `contracts/` records the minimum version per SDK, and CI fails any SDK below it. This is the
  only reliable way to keep four independently released SDKs from drifting apart.

---

## 12. Security

### 12.1 Supply chain

- **STD-SEC-001** — CI **MUST** run in ephemeral hosted runners. **MUST NOT** self-host a
  runner in a path where fork-originated pull requests execute with access to cloud
  credentials. **This is the highest-severity item in this document:** every repo here is
  public with `allow_forking: true`, and a `pull_request` workflow that has AWS credentials
  available lets any stranger run arbitrary code against FFRD infrastructure.
- **STD-SEC-002** — Fork pull requests **MUST** require maintainer approval before any
  workflow runs. **MUST NOT** expose secrets to fork contexts. Required pattern: build/test
  jobs on `pull_request` get no secrets; deploy/publish jobs run on `push` to a tag.
- **STD-SEC-003** — Third-party Actions **MUST** be pinned to a full commit SHA, not a
  mutable tag.
- **STD-SEC-004** — Workflow permissions **MUST** be least-privilege
  (`permissions: contents: read` as default, elevated per job).
- **STD-SEC-005** — Cloud access from CI **MUST** use OIDC federation to short-lived roles.
  **MUST NOT** use long-lived access keys stored as repository secrets.

### 12.2 Plugin runtime

- **STD-SEC-010** — Credentials **MUST** arrive via the manifest `credentials` block resolved
  from the compute provider's secret store. **MUST NOT** be baked into an image, committed in a
  manifest, or logged.
- **STD-SEC-011** — A plugin **MUST** operate with the minimum IAM scope its DataSources
  require; **MUST NOT** request bucket-wide read.
- **STD-SEC-012** — `privileged: true` requires an ADR (§6.4).
- **STD-SEC-013** — Images **MUST** be scanned at build and at deploy; Critical findings block.
  A base image is re-scanned on a schedule — a clean build does not stay clean.

### 12.3 Input handling

- **STD-SEC-020** — **Path traversal.** Payload and manifest DataSource names, `paths` values,
  and `datakey`/`pathkey` values originate outside the plugin. After `{ENV::}`/`{ATTR::}`
  substitution, every path **MUST** be normalized and verified to remain within its declared
  root before any filesystem or object-store operation. `..`, absolute paths, embedded
  null bytes, and symlink escapes **MUST** be rejected. This is the zip-slip/arbitrary-write
  class, and CC's substitution feature is precisely the mechanism that turns a stored path
  template into attacker-influenced input.
- **STD-SEC-021** — **MUST NOT** build a shell command from payload-derived values. Pass an
  argument array. If a wrapped native executable requires a shell, validate against an
  allowlist and reject anything else.
- **STD-SEC-022** — Deserialization **MUST** be limited to the schema-validated JSON used by
  CC. **MUST NOT** use `pickle`, Java native serialization, `yaml.load` without a safe loader,
  or .NET `BinaryFormatter` on any payload- or store-derived bytes.
- **STD-SEC-023** — SQL **MUST** use parameterized statements.
- **STD-SEC-024** — XML **MUST** disable external entity and DTD processing.
- **STD-SEC-025** — Error messages surfaced in logs **MUST** name the failing resource and
  class, and **MUST NOT** leak credentials, full connection strings, stack frames with
  internal hostnames, or bulk data.

---

## 13. Documentation obligations

A repository is incomplete, not done, when code merges without these.

- **STD-DOC-001** — Every repository README **MUST** state: repo class (§1.2), purpose,
  language and minimum version, install/use, build-and-test command, container image name,
  plugin manifest location, supported environment variables, supported payload attributes,
  supported actions, outputs produced, and license. `filesapi`'s README is currently `TEST OF
  V1` while `cc-go-sdk` depends on it; that is a §13 violation on a depended-upon component.
- **STD-DOC-002** — Every plugin **MUST** document its manifest, its inputs/outputs by name,
  its payload attributes (name, type, default, required), and its action list with ordering
  semantics.
- **STD-DOC-003** — Every public function/class **MUST** have API documentation in the
  language's native format. Docstring/comment *style* is `styles.md`; *presence* is a
  standard.
- **STD-DOC-004** — User-facing documentation lives in `cc-home`; implementation detail lives
  in the owning repo. Cross-link, do not duplicate — duplication is how `CONTRIBUTING.rst`
  came to advertise Python 3.5.
- **STD-DOC-005** — Terminology **MUST** match the glossary. **MUST NOT** introduce a synonym
  for an existing term.
- **STD-DOC-006** — Architecture-affecting choices **MUST** have an ADR. Required triggers:
  new dependency in platform/SDK, wire-contract change, storage backend change, `privileged`
  use, deviation from any MUST, deprecation, cross-SDK API decision, security control change.
- **STD-DOC-007** — CHANGELOG **MUST** follow Keep a Changelog and be updated in the same pull
  request as the change.
- **STD-DOC-008** — README **MUST NOT** contain a broken relative link or a reference to a
  repository that is not canonical (§1.3). CI link-checks.
- **STD-DOC-009** — FFRD-facing documentation **MUST** carry the existing `cc-home` caveat that
  it assumes FFRD SOP familiarity and is intended for Federal employees and contractors.
- **STD-DOC-010** — **OPEN DECISION D-7** — Section 508. Proposed: binding on anything a human
  consumes (reports, plots, tables, HTML/PDF output, web UIs). A plugin emitting such an
  artifact **MUST** provide text alternatives for plots, tagged/reading-order-correct
  documents, sufficient contrast, and a non-color-only encoding of result categories.
  Headless data-only plugins are out of scope.

---

## 14. Code review and quality gates

- **STD-REV-001** — No change reaches a protected branch without a pull request and an
  approving review. No self-merge. Direct push to `main` is disabled org-wide.
- **STD-REV-002** — Required checks must pass before merge (table in `README.md` §5). Bypass
  requires a recorded deviation, not a click.
- **STD-REV-003** — **Two-person review** for: platform/SDK code, the wire contract, `conformance/`,
  CI workflow files, anything touching credentials, anything touching Dockerfiles, and any
  deprecation.
- **STD-REV-004** — **MUST NOT** review a pull request that mixes a behavior change with mass
  reformatting or a dependency bump. Separate pull requests; `styles.md` suppressions exist for
  transitional cases.
- **STD-REV-005** — Reviewers **MUST NOT** raise formatting or lint findings — CI enforces those.
  Review attention is for correctness, contract compatibility, determinism, security, tests,
  and clarity.
- **STD-REV-006** — A pull request **SHOULD** be reviewable in one sitting: under ~400 changed
  lines, excluding generated files, lockfiles, goldens, and pure moves. Split by stacked
  branches otherwise.
- **STD-REV-007** — The author **MUST** respond to every review comment, including by
  disputing it. Silently resolving is prohibited.
- **STD-REV-008** — Merged changes are squashed to one commit per pull request with a
  Conventional Commits message, so `main` history reads as a changelog.

---

## 15. Operational standards for FFRD plugins

- **STD-OPS-001** — Every plugin **MUST** register a semantic versioned image tag **and** be
  deployable by digest. `:latest` **MUST NOT** appear in any registered manifest — a
  massively parallel run must be pinned to something.
- **STD-OPS-002** — Every plugin **MUST** document its scaling behavior: embarrassingly
  parallel, intra-plugin parallel (with thread count), or serial; and its memory scaling with
  problem size.
- **STD-OPS-003** — Cost-relevant defaults **SHOULD** be conservative, with the tuning knobs
  named in the README.
- **STD-OPS-004** — A plugin **MUST** be runnable by a developer with only the repo, Docker,
  and `cccli` — no tribal knowledge, no shared cluster. If it cannot, that is a bug (§10.1
  end-to-end tier).
- **STD-OPS-005** — Output layout **MUST** be documented, event-scoped, and collision-safe
  across parallel events (`{ENV::CC_EVENT_NUMBER}` scoping, as in the documented payload
  example). Two events writing the same key is data loss.
- **STD-OPS-006** — **MUST** declare which outputs are required vs optional, so downstream DAG
  nodes fail fast on a missing required output rather than computing on nothing.

---

## Appendix A — Environment variable registry

Seeded from observed usage across `cc-go-sdk`, `cc-py-sdk`, and `cc-home` docs. **OPEN:**
needs maintainer confirmation of exact semantics; this table is the deliverable that makes
§5.4 enforceable.

| Variable | Class | Meaning | Default | Notes |
| --- | --- | --- | --- | --- |
| `CC_STORE_TYPE` | config | Store backend: `S3` \| `FS` | — | SDK selects backend |
| `FSB_ROOT_PATH` | config | Filesystem store root | — | Grandfathered; target `CC_FSB_ROOT_PATH` |
| `CC_LOGGING_LEVEL` | config | Log verbosity | `INFO` | |
| `CC_EVENT_NUMBER` | runtime | Current DAG event index | — | Injected by platform; used in output paths |
| `CC_AWS_ACCESS_KEY_ID` | credential | CC-scoped AWS access key | — | Via `credentials` block only |
| `CC_AWS_SECRET_ACCESS_KEY` | credential | CC-scoped AWS secret | — | Via `credentials` block only |
| `CC_AWS_DEFAULT_REGION` | config | AWS region | — | |
| `CC_AWS_S3_BUCKET` | config | Default bucket | — | |
| `AWS_REGION`, `AWS_S3_BUCKET`, `AWS_S3_ENDPOINT`, `AWS_HTTPS`, `AWS_VIRTUAL_HOSTING` | config | Consumed natively by AWS SDKs | — | Reserved prefix; do not add CC meaning |
| `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | credential | Native AWS SDK credentials | — | Native SDK consumption only |

Rules: see §5.4. Registry changes require a pull request to `cc-standards` plus a
corresponding `cc-home` glossary update.

## Appendix B — Compatibility matrix

Generated by CI from the matrix actually tested; never hand-edited.

| SDK | Canonical repo | Latest | Language floor | Contract version | Last release | Status |
| --- | --- | --- | --- | --- | --- | --- |
| Go | `USACE/cc-go-sdk` *(confirm, D-0)* | pseudo-v `20251024` | 1.22 | `contracts/` @ tag | unpseudo-versioned | **Needs a real tag** |
| Java | `USACE-Cloud-Compute/cc-java-sdk` | v1.1.2 | 17 | | 2026-03 | Active |
| Python | `USACE-Cloud-Compute/cc-py-sdk` | v1.1.0 | 3.11 | | 2025-11 | Active |
| .NET | `USACE/cc-dotnet-sdk` *(confirm, D-0)* | v1.0.5 | net8.0 | | 2024-04 | **Dormant — see D-3** |
| filesapi | `USACE-Cloud-Compute/filesapi` | unpseudo-versioned | 1.22 | | — | **Undocumented; consumed by cc-go-sdk** |

## Appendix C — Frozen wire-format exceptions

Recorded so that no well-meaning contributor "fixes" them (§5.1, STD-WIRE-005).

| Field | Container | Casing | Status |
| --- | --- | --- | --- |
| `payloadAttributes` | payload | camelCase | Frozen |
| `store_name`, `store_type` | payload store/dataSource | snake_case | Frozen |
| `image_and_tag`, `compute_environment`, `retry_attempts`, `execution_timeout`, `linux_parameters`, `privileged` | plugin manifest | snake_case | Frozen |
| `extraHosts` | compute_environment | camelCase | Frozen — inconsistent with its own parent |
| `host_path`, `container_path` | linux_parameters.devices | snake_case | Frozen |

**Do not reformat any of the above.** New fields: camelCase.
