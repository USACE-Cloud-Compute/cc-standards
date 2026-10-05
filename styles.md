# Cloud Compute Style Guide

**Version:** 0.1 (proposed)

**Precedence:** Layer 2. See `README.md` §3.

**Enforced by:** `conformance/` configs. If your formatter and this document disagree, the
config wins and this document is a bug.

**This document covers:** formatting, naming, comments, documentation syntax, tool selection,
configuration of linters.

**This document does NOT cover:** architecture, security, API compatibility, dependency
policy, testing obligations, plugin runtime behavior — all in `standards.md`. Nor branch
naming, review routing, release mechanics — all in `CONTRIBUTING.md`.

---

## 0. How to use this document

**Rule zero: no style debate in pull request review.**

If a finding is in this document, CI already catches it; a review comment is redundant. If a
finding is not in this document, it is not a review comment — it is a change request against
this document. Either way, style is not a review topic. This single rule recovers more
reviewer attention than everything else in this system combined.

Four principles govern every language section:

1. **Language-native style wins.** .NET is PascalCase; Go is not. An org-wide uniform look
   imposed on a language fights its toolchain, its ecosystem, and every contributor arriving
   from outside. Consistency *across* Cloud Compute means "each language looks like itself,
   and every repo in a language looks like every other repo in that language" — never "all
   languages look alike."
2. **The tool is the style.** Every rule here maps to a formatter or linter rule. If a rule
   cannot be mechanized, it does not belong in this document; it belongs in `standards.md` or
   nowhere. Prose style rules that no tool enforces become `CONTRIBUTING.rst`.
3. **Versions live in `conformance/`, not here.** This document deliberately names no tool
   version numbers. Pinning them here would create a second source of truth that rots the
   moment a steward bumps a dependency. Look at `conformance/<lang>/` for what is actually
   enforced today.
4. **Formatting churn is quarantined.** A reformat commit is always its own pull request,
   titled `chore(style): …`, and lands before — never inside — a behavioral change. Reviewers
   may ask for `.git-blame-ignore-revs` entries on large reformats.

**Suppressions.** Every suppression names the rule and gives a reason, in the syntax each tool
accepts:

```python
# ruff: noqa: N806  -- upstream H&H API uses SCREAMING_CASE parameters
```
```go
//nolint:gocyclo // decision table mirrors the published SOP matrix 1:1
```
```java
@SuppressWarnings("unchecked") // TypeToken erasure workaround for generic payload parsing
```
```csharp
#pragma warning disable CA1062 // Called only from validated plugin entry point
```
```markdown
<!-- markdownlint-disable MD013 -- long table rows are atomic -->
```

A bare suppression with no reason is a `standards.md` violation, not a style choice.

---

## 1. Cross-language conventions

These hold in every language because they are about the *codebase*, not the syntax.

### 1.1 Files

- UTF-8, no BOM (except where a toolchain requires it — e.g. some .NET Razor/Win32 cases; in
  this org, none apply).
- LF line endings. `conformance/markdown/.gitattributes` and per-language `.editorconfig`
  /`gofmt` enforce this; do not commit CRLF.
- Exactly one trailing newline; no trailing whitespace.
- Final newline matters: it prevents spurious diffs on append.
- One top-level type per file where the language supports it; file named for that type per the
  language's rule (§2–§5).

### 1.2 Line length

| Language | Soft | Hard | Enforcement |
| --- | --- | --- | --- |
| Python | 88 | 100 | `ruff` `line-length`, formatter owns soft |
| Java | 120 | 140 | Checkstyle `LineLength` |
| Go | — | — | `gofmt` decides; do not configure |
| C# | 120 | — | `.editorconfig` + formatter |
| Markdown | — | — | line length disabled; tables are atomic |
| JSON/YAML | — | — | formatter decides |

Long lines in generated code, golden files, test fixtures, and base64 blobs are exempt —
generated artifacts are never hand-formatted, and reformatting a golden destroys its purpose.

### 1.3 Naming of identifiers

Case conventions are **language-specific** and are set in §2–§5. Two cross-language rules:

- **STD-style:** an identifier that appears on the wire (a JSON key, an environment variable
  name, a payload attribute, an action `type` string, a DataSource name) is **not** a style
  decision. It is part of the contract and is frozen by `standards.md` §5. Never rename a
  wire string to satisfy a linter. See `standards.md` Appendix C for the frozen mixed-casing
  exceptions.
- A language identifier that *carries* a wire string keeps the language's own convention and
  names the wire value inside it: `PayloadAttributes PayloadAttributes` (C#),
  `payloadAttributes` (Python), never a camelCase Python attribute forced to match JSON, nor
  JSON renamed to match Python.

### 1.4 Terminology in identifiers and prose

Match [`cc-home/docs/08_glossary.md`](https://github.com/USACE-Cloud-Compute/cc-home/blob/main/docs/08_glossary.md).
Canonical spellings, and the common drifts to avoid:

| Use | Not |
| --- | --- |
| plugin | job, task, worker, container |
| compute / compute job | run, execution |
| payload | request, config, params |
| manifest | spec, descriptor, definition |
| DataSource / DataStore | bucket, file, dataset |
| Action | step, task, phase |
| Event | scenario, iteration, seed |
| DAG | pipeline, graph, workflow |
| Cloud Compute / CC | CloudCompute, cloudcompute, CCompute, cloud compute |

Note: the repositories themselves drift — `cc-py-sdk`, `cc-python-sdk`, and `cc_py_sdk` all
appear; `Cloud Computeb` appears as a typo in the Go SDK README. Prefer `cc-py-sdk` as the
repo name and `cc-python-sdk` as the PyPI distribution name, and say which you mean.

### 1.5 Comment style

- Comments say **why**, not **what**. `# increment i` is noise.
- A workaround names what it works around and links the issue or ADR.
- `TODO(author): YYYY-MM-DD — description, link` or a tracked issue number. A bare `TODO` is a
  comment-shaped way of not writing an issue. CI flags `TODO` without an owner or link.
- Do **not** commit commented-out code. Delete it; git remembers.
- Do not write block banners that restate the type signature.
- No AI-attribution or pair-programming credits in commit messages or code comments. Attribution
  belongs in `AUTHORS`/`NOTICE` per `standards.md` §3 and in the pull request thread.

### 1.6 Log message style

- Lowercase-ish sentence, no trailing period, no embedded secrets.
- `key=value` context appended: `copying datasource name=%s store=%s bytes=%d`.
- Level discipline: `DEBUG` for developer diagnostics, `INFO` for lifecycle milestones,
  `WARN` for recoverable anomalies, `ERROR` for failures that end the run.
  `INFO` must stay readable at fleet scale: one line per plugin lifecycle milestone, not one
  per record. If it is only useful when something breaks, it is `DEBUG`.
- See `standards.md` §6.3 for where logs go and what they must contain.

---

## 2. Python (`cc-py-sdk`, Python plugins)

### 2.1 Toolchain

| Purpose | Tool | Config |
| --- | --- | --- |
| Format + lint | **Ruff** (single tool for both) | `conformance/python/ruff.toml` |
| Type check | **mypy** (`--strict` in SDKs) | `conformance/python/mypy.ini` |
| Tests | **pytest** | `pyproject.toml` |
| Coverage | `pytest-cov` | `codecov.yml` |
| Dependency management | **uv** (preferred) or `pip-tools` | `uv.lock` committed |

Ruff replaces the flake8/isort/black/pydocstyle stack. `cc-py-sdk` still carries a
cookiecutter-era `Makefile` invoking `flake8` and `tox` with Python 3.5–3.8; migrating it to
Ruff is a Phase 1 task, and until then the repo's own lint is not the standard's lint.

### 2.2 Formatting

- Ruff formatter (Black-compatible): double quotes, magic trailing commas, 4-space indent.
- Line length per §1.2.
- Parenthesized implicit continuation over backslashes, always.
- Type hints required on all public functions in SDK and platform code; `from __future__ import
  annotations` in modules targeting older interpreters.

### 2.3 Naming

| Element | Convention | Example |
| --- | --- | --- |
| Module | `snake_case` | `plugin_manager.py` |
| Class | `PascalCase` | `PluginManager` |
| Function/method | `snake_case` | `get_input_data_source` |
| Constant | `UPPER_SNAKE` | `DEFAULT_LOG_LEVEL` |
| Private | leading `_` | `_resolve_store` |
| Type variable | descriptive, `PascalCase` | `StoreT` |

Existing SDK API uses `snake_case` methods (`get_payload`, `copy_file_to_local`,
`get_attribute_or_fail`) — correct for Python. **Do not** rename these to camelCase for
cross-SDK visual symmetry: that breaks every existing plugin (`standards.md` §11).

### 2.4 Docstrings

- **Google style** for args/returns/raises. Consistent org-wide; do not mix with reST.
- One-line summary in imperative mood ("Return the …", not "Returns" or "This method returns"),
  ending in a period, ≤ 72 chars, blank line after.
- Args/Returns/Raises only where the name is insufficient. Do not document an obvious `str name`.
- Module docstring states purpose and, for plugin entry points, the payload attributes consumed.
- Anything appearing on PyPI **MUST** have docstrings — they are the published documentation.

```python
def get_input_data_source(self, name: str) -> DataSource:
    """Return the input DataSource with the given payload name.

    Args:
        name: DataSource name as declared in the payload's ``inputs``.

    Raises:
        DataSourceNotFoundError: No input DataSource has this name.
        PayloadNotLoadedError: The payload has not been loaded yet.
    """
```

### 2.5 Ruff rule groups enabled

`E`, `F`, `W`, `I` (isort), `N` (pep8-naming), `UP` (pyupgrade), `B` (bugbear), `SIM`, `C4`,
`DTZ` (naive-datetime rejection — directly supports `standards.md` §10.2), `S` (bandit, relaxed
in tests), `RET`, `ARG`, `PTH` (prefer `pathlib` — relevant to `standards.md` §8 path handling),
`RUF`. `ANN` enforced in SDK/platform only.

### 2.6 Python-specific prohibitions

- **MUST NOT** use mutable default arguments.
- **MUST NOT** bare `except:` or `except Exception: pass`. Catch narrow; log or re-raise.
- **MUST NOT** use `pickle` on payload/store bytes (`standards.md` §12.3).
- **MUST NOT** use `yaml.load` without `Loader=yaml.SafeLoader`.
- **MUST NOT** use `datetime.now()` / `date.today()` without an explicit `tz`; `DTZ` enforces.
- **SHOULD** use `pathlib.Path` over `os.path` in new code.
- **MUST** use `match`/`case` or a dispatch table for action routing, not an `if`/`elif` chain on
  `action.type` that silently falls through — an unknown action type must be an explicit error,
  matching the `case _: raise` shape already shown in the SDK README.
- **MUST** use `with` for every file, store, and HTTP resource.
- **MUST NOT** print directly; use the `CC_LOGGING_LEVEL`-honoring logger (`standards.md` §6.3).

---

## 3. Java (`cc-java-sdk`, Java plugins/runners)

### 3.1 Toolchain

| Purpose | Tool | Config |
| --- | --- | --- |
| Format | **Spotless** | `conformance/java/spotless.gradle` |
| Style lint | **Checkstyle** | `conformance/java/checkstyle.xml` |
| Bug patterns | **SpotBugs** (+ `spotbugs-security`) | Gradle plugin |
| Tests | **JUnit 5** | Gradle |
| Coverage | JaCoCo | threshold in Gradle |
| Build | **Gradle** (matches `gradle build -x test` / `gradle shadow` already documented) | `gradle.lockfile` committed |

### 3.2 Formatting

- Google Java Format (AOSP variant if 4-space indent is preferred — pick one, it is in the
  Spotless config, and the choice is not up for debate in review).
- 120-column limit, 4-space indent, no tabs.
- Import order: static first, then all others, no wildcard imports. Wildcard imports are a
  Checkstyle error, not a warning.
- One public top-level type per file.

### 3.3 Naming

| Element | Convention | Example |
| --- | --- | --- |
| Package | all lowercase, no underscores | `mil.army.usace.cc.plugin` |
| Class/interface/record | `PascalCase` | `PluginManager` |
| Method/field | `camelCase` | `getInputDataSource` |
| Constant | `CONSTANT_CASE` | `DEFAULT_LOG_LEVEL` |
| Type parameter | single letter or `ThingT` | `T`, `StoreT` |
| Test class | `ThingTest` | `PluginManagerTest` |

### 3.4 Documentation

- **Javadoc** on every public type and method in the SDK — it is the published API surface.
- First sentence is a standalone summary, imperative mood ("Returns the …").
- `@param` for non-obvious parameters, `@return`, `@throws` for checked exceptions callers must
  handle. Omit `@param` where the name is self-evident; Checkstyle flags the imbalance either way.
- `@implNote` for SDK-internal behavior worth surfacing (e.g. store-selection rules).
- No Javadoc on `private` members unless the logic is genuinely non-obvious.

### 3.5 Java-specific prohibitions

- **MUST NOT** use `System.out` / `System.err` in SDK or plugin code; use the logging facade
  (SLF4J API, binding chosen by the plugin).
- **MUST NOT** call `System.exit()` in SDK library code (`standards.md` §7).
- **MUST NOT** use Java native serialization on payload/store data; JSON only.
- **MUST NOT** use raw types; generics are checked by `-Xlint:all`.
- **MUST** declare `throws` rather than wrap-and-log-and-return-null.
- Prefer records and immutability for payload/manifest data classes; these types are parsed from
  wire data and must not be quietly mutated mid-run.
- `java.util.Optional` for return values that may legitimately be absent
  (`getAttributeOrDefault` semantics); never for fields or parameters.

---

## 4. Go (`cc-go-sdk`, `filesapi`, `cloudcompute`, Go plugins)

### 4.1 Toolchain

| Purpose | Tool | Config |
| --- | --- | --- |
| Format | **gofmt** (`gofumpt` for stricter) | not configurable, by design |
| Import order | **goimports** | grouped std/external |
| Lint | **golangci-lint** | `conformance/go/golangci.yml` |
| Vuln scan | **govulncheck** | CI |
| Tests | `go test ./...` (+ `testify` where already used) | race detector on |
| Docs | `go doc` / pkg.go.dev | — |

### 4.2 Formatting

`gofmt` is not a style preference; it is the language's rule. It is not configurable and is not
arguable. `gofumpt` adds opinionated tightening — adopt it only org-wide, or not at all; a mix
means contributors' editors fight the build.

### 4.3 Naming

| Element | Convention | Example |
| --- | --- | --- |
| Package | short, lowercase, singular, no underscore | `manager`, `store` |
| Exported identifier | `PascalCase` | `PluginManager` |
| Unexported | `camelCase` | `resolveStore` |
| Receiver | 1–2 letters, consistent per type | `func (pm *PluginManager) …` |
| Constant | `camelCase`/`PascalCase` | `defaultLogLevel` |
| Interface | method name + `-er` | `Reader`, `Store` |
| Test file | `foo_test.go` | `manager_test.go` |
| Table test case | descriptive string | `"missing attribute is fatal"` |

Initialisms stay consistent in case: `ID`, `URL`, `JSON`, `S3`, `DAG`, `IO` —
`DataSource.ID`, not `DataSource.Id`; `IOManager`, not `IoManager`. `new`, `make`, `len`, `cap`,
`close`, `copy` remain reserved — do not shadow them.

### 4.4 Documentation

- Every exported identifier has a doc comment beginning with its own name:
  `// PluginManager resolves payloads and provides store access.`
- Package comment in `doc.go` for any non-trivial package, stating purpose and the payload
  attributes/store types it handles.
- Comments explain invariants and error conditions, not mechanics.
- `internal/` marks non-API. `standards.md` §7 requires the export surface be explicit — in Go
  that means lower-case-by-default and deliberate exports only.

### 4.5 Go-specific prohibitions

- **MUST** check every error. `errcheck` is on; `_ = err` requires a reason comment.
- Error strings: lowercase, no trailing punctuation, no `fmt.Errorf("error: …")` prefix
  stutter. Wrap with `%w` so callers can use `errors.Is`/`errors.As`.
- Sentinel errors or typed errors for conditions callers must branch on
  (`DataSourceNotFoundError`), `%v` wrapping for incidental context.
- **MUST NOT** `panic` in library code paths (`standards.md` §7).
- **MUST** honor `context.Context`: first parameter, named `ctx`, never stored in a struct, never
  nil. This is what makes `execution_timeout` and `SIGTERM` handling real.
- **MUST NOT** use `interface{}`/`any` where a concrete type is known; the payload is genuinely
  heterogeneous (`map[string]any`) and that is fine, but do not widen further than the wire
  format requires.
- **MUST** run tests with `-race` in CI.
- `golangci-lint` enabled: `govet`, `errcheck`, `staticcheck`, `unused`, `gosimple`,
  `ineffassign`, `gofmt`/`gofumpt`, `goimports`, `revive`, `gocyclo`, `contextcheck`,
  `nilerr`, `bodyclose`, `errorlint`, `exhaustive`, `gci`, `depguard`.
  `exhaustive` matters for action-type `switch` statements: a new action type must fail the
  build rather than hit a silent default.
- `depguard` enforces the `standards.md` §9 license/dependency allowlist mechanically.

---

## 5. C# / .NET (`cc-dotnet-sdk`, `Usace.CC.Plugin`, .NET plugins)

### 5.1 Toolchain

| Purpose | Tool | Config |
| --- | --- | --- |
| Format | **`dotnet format`** | `.editorconfig` |
| Analyzers | **.NET analyzers / `Microsoft.CodeAnalysis.NetAnalyzers`** | `Directory.Build.props` |
| Style enforcement | `EnforceCodeStyleInBuild` | `Directory.Build.props` |
| Tests | **xUnit** | — |
| Coverage | coverlet | threshold in props |
| Packages | NuGet (GitHub Packages feed) | `packages.lock.json` committed |

`conformance/dotnet/Directory.Build.props` is imported by every repo, so settings are inherited
rather than copied — copying is how four SDKs drift.

### 5.2 Formatting (`.editorconfig`)

- 4-space indent, no tabs; 120-column soft limit.
- `csharp_new_line_before_open_brace = all` (Allman braces — this is the point where .NET
  visibly differs from Go/Java/Python, and that is intentional per principle 1).
- `this.` qualification off; `var` where the type is apparent.
- `System.*` usings first, then third-party, then project; `using` directives outside the
  namespace body.
- Nullable reference types **enabled** in new code; `<TreatWarningsAsErrors>true</TreatWarningsAsErrors>`
  for SDK/platform projects. Given `cc-dotnet-sdk`'s dormancy, enabling nullable is a staged
  change — do it project-by-project, each its own pull request.

### 5.3 Naming

| Element | Convention | Example |
| --- | --- | --- |
| Namespace | `PascalCase`, rooted `Usace.CC.*` | `Usace.CC.Plugin.Manager` |
| Type | `PascalCase` | `PluginManager` |
| Method/Property | `PascalCase` | `GetInputDataSource` |
| Public field | — avoid; **MUST NOT** add new ones | |
| Parameter | `camelCase` | `dataSourceName` |
| Private field | `_camelCase` | `_session` |
| Constant | `PascalCase` | `DefaultLogLevel` |
| Interface | `I` prefix | `IDataStore` |
| Enum member | `PascalCase` | `StoreType.S3` |
| Async method | `…Async` suffix | `CopyFileToRemoteAsync` |
| CancellationToken | last parameter | |
| Test class | `ThingTests` | `PluginManagerTests` |

Existing SDK names (`GetAttributeOrDefault`, `GetAttributeOrFail`, `CopyFileToRemote`,
`GetStore`) already follow this — keep them. Note the existing namespace root is
`Usace.CC.Plugin`; keep the `Usace` casing rather than `USACE` in C# identifiers, since all-caps
segments break PascalCase tooling conventions. (The NuGet feed path uses `USACE`; that is a URL,
not an identifier, and is frozen.)

### 5.4 Documentation

- **XML doc comments** (`///`) on all public and protected members; `GenerateDocumentationFile`
  true and `CS1591` treated as an error in SDK projects, which makes missing docs a build
  failure rather than a review comment.
- `<summary>` imperative ("Returns the …"), `<param>`, `<returns>`, `<exception cref="…">` for
  documented throws.
- `<remarks>` for store-selection and substitution semantics — behavior that a plugin author
  must know but that does not fit one line.

### 5.5 C#-specific prohibitions

- **MUST NOT** use `BinaryFormatter` (also obsolete-ineligible for security reasons).
- **MUST NOT** use `.Result` or `.Wait()`; `ConfigureAwait(false)` in library code.
- **MUST NOT** swallow exceptions; no empty `catch`.
- **MUST** honor `CancellationToken` throughout any I/O path.
- Prefer `sealed` classes and immutable records for parsed wire types.
- **MUST NOT** `Console.WriteLine` in SDK code; `ILogger` or the SDK logging facility.
- Explicit `TargetFramework` pinned per `standards.md` §2; no `netX.0` wildcard.

---

## 6. Configuration-as-code style

### 6.1 JSON (manifests, payloads)

- 4-space indent, matching the examples already in `cc-home/docs/05_plugin-manifest.md` — the
  docs and the fixtures must look alike or copy-paste produces churn.
- Keys sorted by semantic grouping, not alphabetically: identity first (`name`,
  `image_and_tag`), then resources, then environment, then credentials, then optional
  behaviors. Preserve the documented field order in templates.
- **Never** reformat an existing committed manifest or golden: whitespace changes to a golden
  invalidate diffs (`standards.md` §10.2).
- No comments in shipped JSON. Where annotation is needed, use a sibling `.md`, or a
  `_comment` key only in fixtures that are never registered.
- No trailing commas (JSONC is not accepted by the CLI).
- Secrets: placeholder or `secretsmanager:<name>::` reference only (`standards.md` §4).

### 6.2 YAML (GitHub Actions, `compute.json` where YAML)

- 2-space indent, no tabs. Never 4 in Actions — consistency with the ecosystem matters more
  than the extra level.
- Quote nothing that does not require quoting, **except** version-like scalars and anything that
  YAML may coerce (`"no"`, `"on"`, `"3.10"`, `"007"`). `on:` is the classic trap: unquoted
  `on: {push: ...}` collapses to a boolean key.
- Pin actions to full commit SHA (`standards.md` §12.1).
- Prefer `run: |` blocks with a single logical command per line; a shell one-liner in YAML is
  unreviewable.
- `actionlint` enforces.

### 6.3 Dockerfile

- Lowercase instructions (`from`, `run`, `copy`) — matches the existing examples in `cc-home`;
  pick case once, stay with it.
- `# syntax=docker/dockerfile:1` on the first line.
- Multi-stage: `AS builder` then a minimal `AS prod`, per the established pattern.
- Order for cache efficiency: metadata → package install → dependency manifests → dependency
  install → source copy → build. Putting `COPY . .` before dependency install invalidates the
  cache on every edit.
- Pin every base image by tag; add `# updated YYYY-MM-DD` when bumping.
- `WORKDIR` absolute; `ENTRYPOINT` exec form (`["java","-jar","x.jar"]`), never shell form.
- Drop to non-root (`USER`) before `ENTRYPOINT` (`standards.md` §6.4).
- Label block at the top so provenance is visible without building.
- `hadolint` enforces; DL3008/DL3009 (pin apt packages) may be disabled only with a reason
  comment, since unpinned native packages are exactly the `standards.md` §9 hazard.

### 6.4 Markdown

`markdownlint` with `markdown/markdownlint.yaml`:

- ATX headings (`#`), no setext except the document title in long-form docs.
- One sentence per line in prose. Diffs become word-level and review comments anchor to a
  stable line. Table rows stay on one line (hence the MD013 exemption).
- `setext_with_atx` heading style, fenced code blocks with language tags always — an untagged
  fence gets no highlighting and no shell-linting.
- Lists: `-` for unordered, `1.` for ordered (let renderers number).
- Line length unlimited (tables).
- Link style: reference links in documents with many links; inline elsewhere. Relative links for
  in-repo targets, absolute for cross-repo — and CI link-checks both (`standards.md` §13).
- Every fenced block **MUST** declare a language. `bash` for commands, `json` for manifests, and
  no `text` where a real language applies.
- Tables: pipes aligned, header row required, no HTML unless the layout is impossible in
  Markdown.
- Filename: `kebab-case.md`, except the GitHub-recognized names (`README.md`, `CONTRIBUTING.md`,
  `SECURITY.md`, `CODE_OF_CONDUCT.md`, `CHANGELOG.md`, `LICENSE`), which keep their fixed casing.

### 6.5 Commit messages and PR titles

Conventional Commits, enforced on the PR title (`standards.md` §14, squashed merges):

```
<type>(<scope>): <imperative summary, ≤ 72 chars>

<body: what and why, wrapped at 100>

Fixes #<issue>
```

Types: `feat`, `fix`, `perf`, `refactor`, `docs`, `test`, `chore`, `build`, `ci`, `style`,
`revert`. `BREAKING CHANGE:` footer or a `!` after the type forces a major version
(`standards.md` §11) — this is what makes automated release notes honest.

Scope is the component: `payload`, `manifest`, `store`, `s3`, `fsb`, `cli`, `sdk-py`, `sdk-java`,
`sdk-go`, `sdk-dotnet`, `substitution`, `actions`, `docs`. A change to `payload` or `substitution`
automatically requests the two-person review defined in `standards.md` §14.

Examples:

```
feat(sdk-go): accept optional outputs in payload validation
fix(store): sort listing keys before reducing to keep aggregation deterministic
feat(substitution)!: reject empty attribute values instead of substituting ""

BREAKING CHANGE: {ATTR::x} resolving to an empty string now fails at load per
STD-WIRE-011. Previously it produced an empty path segment.

Fixes #142
```

---

## 7. Adding a language

The system is designed to grow. A new language (Rust, TypeScript, Julia) requires, in order:

1. An ADR (`standards.md` §13, STD-DOC-006).
2. A `conformance/<lang>/` directory: formatter, linter, build, test, coverage, and the
   vulnerability scanner, each pinned.
3. A `styles.md` section following this template — **in this order**, because the tool table
   comes first:

   | Subsection | Content |
   | --- | --- |
   | Toolchain | Table: purpose → tool → config file |
   | Formatting | What the formatter decides; the column limit |
   | Naming | Table by element, with a real example from this codebase |
   | Documentation | Doc-comment format and enforcement mechanism |
   | Lint rule set | Which rule groups are on and why each non-default one matters here |
   | Prohibitions | MUST NOTs specific to the language's footguns |

4. An implementation of `contracts/` conformance fixtures. **A language is not an official SDK
   language until it passes them** (`standards.md` §7).
5. A language steward named in `CODEOWNERS`.
6. `ci/reusable-lint.yml` gains the language key.

Rules for the section itself: state only what the tools enforce. Any sentence that describes
taste rather than configuration is out of scope for this document.
