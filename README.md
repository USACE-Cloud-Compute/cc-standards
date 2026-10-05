# cc-standards

**Status:** Proposed — for inclusion in the FFRD Software Development Strategy
**Owner:** CC Platform Maintainers (see `CODEOWNERS`)
**Applies to:** All repositories in the `USACE-Cloud-Compute` organization, and to plugin
repositories authored by FFRD program contractors that are registered against a Cloud
Compute (CC) environment.

This file describes the *plan*. It does not itself contain rules. It says which document owns
which question, in what order they override each other, who maintains them, and how they
get enforced. Read this first; then read `standards.md`.

```
cc-standards/
├── README.md                      # this plan: ownership, precedence, rollout
├── standards.md                   # WHAT must be true. Language-agnostic. Reviewable.
├── styles.md                      # HOW it looks. Language-specific. Tool-enforced.
├── CONTRIBUTING.md                # PROCESS: how a change gets from your head to main.
├── SECURITY.md                    # Vulnerability disclosure (GitHub-recognized filename)
├── CODE_OF_CONDUCT.md             # Adopted code of conduct (GitHub-recognized filename)
├── CODEOWNERS                     # Review routing for the standards repo itself
├── LICENSE                        # MIT, matching cloudcompute
├── adr/
│   ├── README.md                  # ADR index + acceptance criteria
│   ├── ADR-0000-record-architecture-decisions.md
│   └── templates/adr_template.md
├── templates/                     # copied into repos at scaffold time
│   ├── pull_request_template.md
│   ├── issue_bug.md
│   ├── issue_feature.md
│   ├── repo_README.md
│   └── repo_CONTRIBUTING.md       # 15-line stub that points back here
├── conformance/                   # THE ENFORCEMENT PACK (machine-readable)
│   ├── python/ruff.toml
│   ├── python/mypy.ini
│   ├── java/spotless.gradle
│   ├── java/checkstyle.xml
│   ├── go/golangci.yml
│   ├── dotnet/.editorconfig
│   ├── dotnet/Directory.Build.props
│   ├── markdown/markdownlint.yaml
│   ├── docker/hadolint.yaml
│   └── actions/actionlint.yaml
├── contracts/                     # CROSS-SDK PARITY FIXTURES
│   ├── payload/                   # golden payload JSON + expected parse results
│   ├── manifest/                  # golden plugin/compute manifests
│   ├── substitution/              # {ENV::} / {ATTR::} cases incl. failure modes
│   └── conformance_suite.md       # what each SDK must pass to claim conformance
└── ci/
    ├── reusable-lint.yml          # reusable workflow: all languages
    ├── reusable-contracts.yml     # runs contracts/ against the calling SDK
    └── reusable-release.yml
```

### What each document owns

| Question | Owner document |
| --- | --- |
| May this dependency be used? | `standards.md` |
| Is a breaking change to the payload allowed? | `standards.md` |
| What must a plugin's exit code communicate? | `standards.md` |
| Indentation, quotes, import order, docstring format, lint rules | `styles.md` |
| Should this be a class or a module? | `standards.md` |
| How do I open a pull request? | `CONTRIBUTING.md` |
| Who reviews this, and how long do they have? | `CONTRIBUTING.md` + `CODEOWNERS` |
| What does `CC_STORE_TYPE` mean? | `standards.md` Appendix A (environment registry), mirrored in the `cc-home` glossary |
| How do I use the Python SDK? | That SDK's own `README.md` / API docs |
| What is a DataSource? | `cc-home/docs/08_glossary.md` — **the glossary stays the single source of terminology.** `cc-standards` links to it and never redefines a term. |

### Explicit non-goals

- `standards.md` does not specify formatting. Full stop.
- `styles.md` does not make architectural, security, or API-compatibility judgments.
- `CONTRIBUTING.md` does not restate rules from `standards.md`. It sequences them.
- `cc-standards` does not hold product documentation. Tutorials and concept docs remain in
  `cc-home`, so there is one place a new developer is sent to learn CC.

---

## Precedence

Where two documents disagree, the higher layer wins. A lower-layer document may tighten a
rule but never relax it.

```
Layer 0  Federal & DoD/USACE policy
         FAR / software assurance (OMB M-22-18, M-23-16) · Section 508 ·
         export control · records management · the FFRD SOP
         → NON-WAIVABLE. Not by an ADR, not by a maintainer, not by a deadline.

Layer 1  cc-standards/standards.md
         Language-agnostic, testable MUST/SHOULD obligations.

Layer 2  cc-standards/styles.md + conformance/ pack
         Mechanical. Machine-decided. Not arguable in review.

Layer 3  cc-standards/CONTRIBUTING.md
         Process gates that move code into a repository.

Layer 4  Repository-local config and docs
         Narrow, declared deviations.

Layer 5  Team convention
         Everything unwritten. Free to vary. Must never contradict Layers 0–4.
```

### RFC 2119 keywords

`standards.md` and `styles.md` use **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**,
**MAY** with their RFC 2119 meanings. This is normative, not decorative:

- **MUST** — non-conformance blocks merge. A CI check enforces it, or one is scheduled in
  the enforcement table.
- **SHOULD** — non-conformance requires a recorded justification in the pull request
  description. Reviewer decides.
- **MAY** — genuinely optional.

If a sentence cannot be classified as one of those three, it is documentation and belongs
in `cc-home`, not here.

---

## Deviations

Three mechanisms, deliberately unequal in cost. Escalating cost is the point.

**1. Inline suppression.** For Layer 2 only. Requires the rule identifier and a reason.

```python
# ruff: noqa: N806  -- upstream H&H API uses SCREAMING_CASE parameters
```

A bare `# noqa`, `// nolint`, `@SuppressWarnings`, `#pragma warning disable`, or
`// NOLINT` with no reason is itself a `standards.md` violation and is rejected in review.

**2. Repository-level deviation.** A `CC-DEVIATIONS.md` at the repo root. Each entry:
clause reference, what is relaxed, scope, reason, expiry date, approving maintainer.
Deviations without an expiry date are invalid and the CI check `deviation-lint` fails them.
Recommended default expiry: 180 days, renewable once.

**3. Architecture Decision Record.** For anything affecting more than one repository, the
wire contract, a dependency, or Layer 1. Permanent, immutable once accepted, superseded
rather than edited. ADRs are the only mechanism that can bind all four SDKs at once, and
they are the audit trail the Software Development Strategy needs to show *why* a standard
changed.

**Layer 0 may not be deviated by any of the three.**

---

## Enforcement map

Standards without checks are preferences. Each MUST in `standards.md` must appear here with
a named check. A MUST with no check and no scheduled check is a drafting defect in the
standard.

| Area | Check | Mechanism | Scope | Blocking? |
| --- | --- | --- | --- | --- |
| Formatting | `format-check` | ruff format / spotless / gofmt / dotnet format | PR | Yes |
| Lint | `lint` | ruff / Checkstyle+SpotBugs / golangci-lint / .NET analyzers | PR | Yes |
| Type/compile | `build` | mypy --strict (SDK), javac, go vet, Roslyn nullable | PR | Yes |
| Unit tests | `test` | pytest / JUnit 5 / go test / dotnet test | PR | Yes |
| Coverage floor | `coverage` | Per-language coverage tool vs `codecov.yml` threshold | PR | Yes (SDK ≥ 80%, plugin ≥ 70%) |
| Cross-SDK contract parity | `contracts` | `ci/reusable-contracts.yml` against `contracts/` | SDK PR + nightly | Yes |
| Secret exposure | `secret-scan` | gitleaks (or org-approved equivalent) | PR + push | Yes |
| Dependency vulnerabilities | `dependency-audit` | `govulncheck`, pip-audit, OWASP dependency-check, NuGet audit | PR + daily | Yes on Critical/High |
| License policy | `license-check` | License checker over lockfiles/SBOM against allowlist | PR | Yes |
| SBOM present | `sbom` | Syft/CycloneDX attached to release artifact | Release | Yes |
| Image hygiene | `image-scan` | Trivy/Grype on built plugin image | PR + nightly | Yes on Critical |
| Manifest validity | `manifest-validate` | JSON Schema validation of every `*.manifest.json` | PR | Yes |
| Deviation expiry | `deviation-lint` | Parses `CC-DEVIATIONS.md` for expired/undated entries | PR | Yes |
| Commit/PR convention | `pr-title` | Conventional Commits lint on PR title | PR | Yes |
| Changelog updated | `changelog` | Diff check against `CHANGELOG.md` | PR | Yes (SDK, CLI) |
| DCO sign-off | GitHub DCO | `Signed-off-by` trailer | PR | Yes (external forks) |

Implementation: `ci/reusable-lint.yml` is a reusable workflow. One repo adds five lines and
inherits every check. This is the highest-leverage item in the whole plan — it converts
"27 repos each inventing their own pipeline" into one maintained pipeline with per-repo
language toggles.

```yaml
# .github/workflows/ci.yml in any consuming repo
name: ci
on: [pull_request]
jobs:
  cc:
    uses: USACE-Cloud-Compute/cc-standards/.github/workflows/reusable-lint.yml@v1
    with:
      languages: '["python"]'
      coverage_min: 80
```

---

## Ownership and maintenance

| Role | Holds | Duties |
| --- | --- | --- |
| **Standard Owner** (1, CC platform lead) | Veto on Layer 1 | Approves MUST-level changes and ADRs. Signs the annual review. |
| **CC-standards maintainers** (2–3) | Merge rights in `cc-standards` | Triage change requests, keep `conformance/` current, run the quarterly drift review. |
| **Language stewards** (1 per SDK) | Layer 2 in their language | Own toolchain version bumps for their SDK. A version bump is a minor release, never bundled with a feature. |
| **Repo maintainers** | Layer 4 | Own repo-local deviations and their expiry. |
| **FFRD program office** | Layer 0 | Confirms policy alignment; owns the SOP interface. |
| **Every contributor** | — | Files a change request when they hit an ambiguity. Ambiguity is a defect. |

### Cadence

| Activity | Frequency | Output |
| --- | --- | --- |
| Standards change requests triaged | Biweekly | Accepted / deferred / rejected with reason |
| Drift review: does each document match reality? | Quarterly | Doc corrections, or the `CONTRIBUTING.rst` outcome repeats |
| Toolchain version refresh (ruff, golangci-lint, analyzers) | Quarterly | Minor release of the conformance pack |
| Full standards review against the codebase | Annually | Version bump of `cc-standards` |
| SDK compatibility matrix refresh | Each SDK release | Updated matrix, `standards.md` §7 |

### Versioning this system itself

`cc-standards` is a dependency with real blast radius: a tightened rule breaks builds in 27
repos. Therefore:

- Tagged releases, consumed **by tag** (`@v1.4.0`), never `@main` or `@v1`. Floating refs
  make CI non-reproducible, which is unacceptable in a program that must reproduce FFRD
  hydrologic results.
- Rule additions are a **minor** release. Removals, or flips from SHOULD to MUST, are
  **major** with a 90-day notice.
- `conformance/` follows the same tags. A repo pins a conformance version in
  `.cc-standards-version` and upgrades by PR, so a toolchain change never lands as
  unrelated churn inside a feature pull request.
- Every MUST carries a stable identifier (e.g. `STD-WIRE-001`) so a PR, a CI message, and a
  deviation entry can all cite one thing.

---

## Anti-goals

Deliberately refused, because these are how standards programs fail:

- **No org-uniform style imposed on a language.** .NET is PascalCase, Go is not. Forcing
  uniformity fights the toolchain, the ecosystem, and every new contributor. `styles.md`
  principle 1 makes language-native style win.
- **No style debate in pull request review.** If it is in `styles.md`, the formatter decides
  and the reviewer moves on. If it is not in `styles.md`, it is not a review comment — it is
  a change request against `styles.md`.
- **No prose-only rules.** 
- **No big-bang rewrite of 27 repos.**
- **No standards repo that cannot be built.** `cc-standards` holds itself to the same CI, and
  `conformance/` configs are validated by the tools that read them, in CI, on every PR here.
- **No second glossary.** Terminology lives in `cc-home/docs/08_glossary.md`.

---

## Reading order for a new developer

1. `cc-home/docs/01_cc-for-dummies.md` — concepts
2. `cc-home/docs/08_glossary.md` — vocabulary
3. `cc-home/tutorials/hello-world/README.md` — first run
4. `cc-standards/CONTRIBUTING.md` — first pull request
5. `cc-standards/standards.md` — the obligations
6. `cc-standards/styles.md`, your language section — the mechanics

FFRD program developers additionally read the FFRD SOP first, per the note already in the
`cc-home` README about `/tutorials/FFRD` assuming SOP familiarity.

