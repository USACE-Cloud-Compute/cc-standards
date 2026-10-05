# Contributing to Cloud Compute

**Version:** 0.1 (proposed) · **Precedence:** Layer 3. See `README.md` §3.

This document is the **process**: how a change gets from your head into a protected branch.
It does not restate rules — those are in `standards.md` (obligations) and `styles.md`
(mechanics). Where this document and those disagree, they win.

**Table of contents**
1. Who may contribute · 2. Before you start · 3. Issues · 4. Setup · 5. Branches ·
6. **Fork rules** · 7. Commits · 8. **Pull request rules** · 9. Review · 10. Merge ·
11. Releases · 12. Timelines · 13. Closing a contribution

---

## 1. Who may contribute

Cloud Compute repositories are public. Contributors fall into four lanes, and the lane
determines your path — most importantly whether you use a **fork** or a **branch**:

| Lane | Who | Path |
| --- | --- | --- |
| **A. Core maintainer** | Named in `CODEOWNERS` for the repo | Branch in-repo; can approve and merge |
| **B. Org contributor** | Member of `USACE-Cloud-Compute` (Federal employee, or contractor added to the org) | **Branch in-repo. Do not fork.** |
| **C. Program contributor** | FFRD contractor/LAN employee not yet in the org | Request org membership, or fork + PR |
| **D. External** | Anyone outside USACE/FFRD | Fork + PR; code changes gated (see §6.4) |

**OPEN DECISION D-1** — whether lane D code changes are accepted at all. Draft position:
issues and documentation from lane D are always welcome; code from lane D that touches an SDK
or the wire contract requires an accepted ADR first, so nobody writes a large change we cannot
take.

FFRD program work additionally follows the FFRD SOP and its job aids; where the SOP imposes a
review or approval step, it is **additive** to this process, never a substitute for it.

---

## 2. Before you start

**Open an issue before writing code** if your change is any of:

- more than ~50 lines, or spans files you do not own
- a new feature, a new plugin action, a new CLI command
- any change to a payload, manifest, compute file, or environment variable — i.e. anything in
  `standards.md` §5, regardless of size
- a new dependency, or a dependency major bump
- a breaking change, a deprecation, or a removal
- anything you expect to take more than half a day

An issue costs two minutes and prevents a full review cycle spent on a change that cannot be
accepted. Use the issue templates. Skip the issue only for: a typo, an obvious bug fix with a
linked reproduction, a dependency patch bump, or a documentation correction.

**Check first that the rule you are about to work around is not already answered** in
`standards.md`, `styles.md`, the ADR log, or existing open issues. If it is answered and you
disagree, comment on that thread rather than opening a parallel change.

---

## 3. Issues

| Label | Meaning |
| --- | --- |
| `bug` | Observable behavior differs from documented behavior |
| `contract` | Touches payload/manifest/env wire format |
| `parity` | The four SDKs disagree; a `contracts/` fixture caught divergence |
| `enhancement` | New functionality |
| `docs` | Documentation gap or error |
| `standards-change` | Change request against `cc-standards` itself |
| `deviation` | Requesting or reporting a repo-level deviation |
| `security` | **Do not open publicly — see `SECURITY.md`** |
| `good-first-issue`, `help-wanted` | Claimable without prior discussion |
| `needs-triage`, `needs-decision`, `blocked` | Workflow state |

A bug report **MUST** include: affected repo and version/tag, OS/arch, language and toolchain
version, the exact command, expected vs actual, the full error output, and a minimal
reproduction (a failing test is ideal; a payload file that triggers it is better than prose).

A feature request **MUST** include: the problem in user terms, the proposed change, whether it
touches the wire contract, and its effect on each of the four SDKs. "Add X to the Python SDK"
is incomplete — state what Java, Go, and .NET do instead, or the parity gate will reject it
later (`standards.md` §7).

---

## 4. Local setup

```bash
# 1. Lane B/A: add the upstream as origin and branch from it.
git clone git@github.com:USACE-Cloud-Compute/<repo>.git
# Lane C/D: fork on GitHub, then clone your fork and attach upstream.
git clone git@github.com:<you>/<repo>.git
git remote add upstream git@github.com:USACE-Cloud-Compute/<repo>.git

# 2. Install the pinned toolchain (one file per language, per standards.md §2)
mise install            # or asdf, or the repo's documented equivalent

# 3. Install hooks: formatter + lint + commit-message on commit
make hooks              # or: pre-commit install
```

Before pushing, run the same gate CI runs:

```bash
make ci     # format-check, lint, build, test — one command (standards.md STD-LANG-004)
```

If `make ci` is missing or passes locally while CI fails, that is a bug in the repository —
open an issue. Do not work around it by loosening your local checks.

**Do not** rely on your editor's formatter to match CI; use the config in
`conformance/<lang>/` imported by this repo. Editor settings that disagree with CI produce
commits that fail.

---

## 5. Branch naming

```
<type>/<issue-number>-<short-description>
```

`type` ∈ `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `sec`, `style`,
`release`, `dep`.

```
feat/142-support-optional-outputs
fix/207-s3-listing-order
docs/88-rewrite-contributing
dep/311-aws-sdk-go-v2-2-3000
release/cc-py-sdk-1.2.0
```

- Always include the issue number when one exists — it links CI output, branch, and PR without
  manual cross-referencing.
- Lowercase, hyphens, ASCII, ≤ 60 characters after the slash. No `feature/` spelled out, no
  `WIP`, no personal names, no dates, no ticket-tracker prefixes from other systems.
- `main` is the only long-lived branch. `release/<major>.x` may exist for a supported major
  (`standards.md` §11) and requires an ADR. Long-lived personal branches in-repo are not
  permitted — that is what forks are for.

---

## 6. Fork rules

This is the section most likely to be gotten wrong, so it is explicit.

### 6.1 Do not fork if you can branch

Members of the organization create branches in the repository. Forking when you have write
access creates an unnecessary second copy, loses `CODEOWNERS` review routing on some
operations, and makes cross-repository work harder. **Forking is for people who cannot
branch.**

### 6.2 Fork mechanics

1. Fork on GitHub into your **personal** account or your employer's organization — never into
   another USACE repository, and never into a personal account you do not control long-term.
2. Keep your fork's default branch as `main`, matching upstream. Do not rename it.
3. Sync **by rebase or merge from `upstream/main`, at the start of every work session**:
   ```bash
   git fetch upstream
   git rebase upstream/main      # or: git merge upstream/main
   ```
   A PR that is 40 commits and three weeks behind will be asked to rebase before review.
4. One fork branch per issue. Do not stack unrelated work on one long-lived fork branch and
   PR it as a single change.
5. Never push `upstream/main` to your fork's `main` and then open a PR from it. Always PR from
   a topic branch.
6. Delete merged branches. Stale branches in forks of public repos are how old experimental
   code gets accidentally PR'd back.

### 6.3 What must never go into a fork

**Your fork is public and is not under our control. Deletion does not remove it from caches,
archives, or GitHub's event stream.**

- **No secrets of any kind** — AWS keys, GitHub tokens, connection strings, `.env` files,
  private registry credentials. Not even "temporarily, to test." `standards.md` §4.
- **No PII, PHI, or CUI.** No real post-event survey data, no damage or ownership records, no
  customer-identifying gauge data, no internal-only datasets. `standards.md` §4.
- **No internal hostnames, account IDs, tenant IDs, or network topology.**
- **No proprietary or licensed third-party binaries** (model executables, commercial solvers,
  licensed datasets) unless redistribution is explicitly permitted. A build that needs them
  fetches them from the approved internal mirror at build time — it does not commit them.
- **No unfinished or exploratory FFRD program analysis.**
- **No credentials-bearing `compute.json` or `.env`** — a very common accident in this
  ecosystem, since the CLI is `.env`-driven. Verify `.gitignore` covers `.env*` before your
  first commit, and check `git status` output, not just the diff.

If you accidentally push any of the above to a fork: **treat it as compromised immediately.**
Rotate the credential and contact the maintainers through `SECURITY.md`. Do not assume that
deleting the commit or the fork resolves it.

### 6.4 External contributions (lane D)

- Documentation, tests, issue reports, and reproduction cases: always welcome, normal PR flow.
- Code changes to **plugins**: accepted after standard review.
- Code changes to **SDKs, `cloudcompute`, `cloudcompute-cli`, `filesapi`, the wire contract, or
  `cc-standards`**: require an accepted issue and, for anything non-trivial, an ADR agreed
  before the work starts. The reason is not distrust — it is that these components carry a
  compatibility obligation to every registered plugin image in the program (§8 of
  `standards.md`), and an externally-authored change that silently breaks a consumer is very
  expensive to unwind after release.
- All lane C/D commits **MUST** be signed off (DCO) — §7.
- **OPEN DECISION D-5** — DCO vs CLA. Draft: DCO `-s` for all external commits; IP assignment
  handled by the contract vehicle for lanes B and C.

### 6.5 CI behavior for fork PRs

**Mandatory, non-negotiable** (`standards.md` §12.1, STD-SEC-002):

- Workflows triggered by fork pull requests run with **no secrets** and require maintainer
  approval before first run.
- Build/test in a `pull_request` context never receives cloud credentials.
- Anything needing real credentials (publish, deploy, real-S3 integration) runs on `push` to a
  protected ref or on a `workflow_dispatch` by a maintainer.
- If your change legitimately needs credentialed testing to be reviewable, say so in the PR and
  a maintainer will run it for you. Do not restructure the workflow to route secrets into fork
  contexts; that is the single most dangerous shortcut available in this repository set.

---

## 7. Commit rules

- **Conventional Commits** — `type(scope): summary` (see `styles.md` §6.5). Enforced on the PR
  title, and on commits where a repo uses `commitlint`.
- **Imperative mood, ≤ 72 characters**, no trailing period.
- Body explains *why*, wrapped at 100. Link the issue (`Fixes #142`, `Refs #88`).
- `BREAKING CHANGE:` footer or `!` after the type → major version bump (`standards.md` §11).
- **Sign-off** on every commit from lanes C and D:
  ```bash
  git commit -s -m "fix(store): sort listing keys before reducing"
  ```
  This adds `Signed-off-by:`, asserting you have the right to submit it under the repo's
  license. Do not fabricate it.
- **Author email must be one you control** and, for lanes B and C, one your organization will
  attest to. A mismatched author email makes the contribution unattributable for provenance
  and license records.
- **No AI-attribution trailers, no "Co-Authored-By: <tool>", no generated-by banners** in
  commit messages or file headers. Attribution is a legal/provenance matter handled in
  `AUTHORS`/`NOTICE` and the PR thread, not in git metadata.
- Do not commit: generated build output, IDE settings, local paths, `.env` files, credentials,
  large binaries, reformatted files unrelated to your change.
- Keep each commit buildable. If you bisect `main`, every commit should compile — a
  build-broken intermediate commit destroys bisectability for whoever hits the next regression.

---

## 8. Pull request rules

### 8.1 Opening

- **Title:** Conventional Commit form, ≤ 72 chars, describing the *change*, not the task.
  `fix(payload): reject unknown substitution syntax` — not `fix`, not `wip ticket 207`.
- **Base:** `main`, unless you are targeting a documented `release/<major>.x`.
- **One concern per PR.** Do not mix: a behavior change with a reformat; a dependency bump with
  a feature; a rename with a logic change; an SDK change with the plugin that consumes it
  (sequence those PRs and mark the dependency).
- **Size:** target < 400 changed lines excluding generated files, lockfiles, goldens, and pure
  moves (`standards.md` §14). Larger: split with stacked branches. Unavoidably large
  (a generated-client refresh): say so in the description and provide a guided tour.
- Fill in the PR template completely. `N/A` is an acceptable answer; an empty section is not —
  it tells the reviewer you did not consider the question.
- **The wire-contract section is mandatory.** If your diff touches payload, manifest, compute
  file, or environment variables, you **MUST** state, explicitly:
  1. Which keys/variables are added, changed, or removed.
  2. Whether the change is backward compatible for **already-registered plugin images**.
  3. Whether all four SDKs need the same change, and the tracking issue for each.
  4. Which `contracts/` fixtures were added or updated.
- Attach the local `make ci` result if any check is not green in CI yet, and explain why.
- Mark `Draft` while you are still exploring. Draft PRs are welcome and useful — early
  architectural feedback is cheap, and late architectural feedback is a rejected PR.

### 8.2 Definition of ready

A PR leaves draft when:

- [ ] CI is green, or every failure is explained in the description.
- [ ] Tests added or updated; a bug fix ships a test that fails without the fix.
- [ ] Coverage above the floor for the repo class.
- [ ] `contracts/` fixtures updated if anything in `standards.md` §5 changed.
- [ ] Documentation updated in the same PR: README, docstrings/Javadoc/Godoc/XML docs,
      `CHANGELOG.md`, and the environment registry if variables changed.
- [ ] `Signed-off-by` present on all lane C/D commits.
- [ ] `CC-DEVIATIONS.md` entry added, with expiry, for any MUST you are suppressing.
- [ ] ADR attached if a §13 trigger applies.
- [ ] Branch rebased on current `main`.
- [ ] No secrets, no CUI/PII/PHI, no internal hostnames in the diff **or the PR description or
      attached logs** — the PR is public too.

### 8.3 Definition of done

- [ ] All `CODEOWNERS` reviews obtained; two-person review where §14 of `standards.md` requires.
- [ ] Every review comment addressed or explicitly disputed in-thread.
- [ ] All conversations resolved by the author, not silently by the reviewer.
- [ ] No `needs-decision` or `blocked` labels remaining.
- [ ] Release notes entry present if the change is user-visible.
- [ ] Downstream tracking issue opened for any other SDK needing the same change — a parity
      change merged into one SDK without the others tracked is a divergence in waiting.
- [ ] Squash-merged with a Conventional Commit message that reads well in `main`'s history.

---

## 9. Review

**Routing.** `CODEOWNERS` assigns a review request automatically. Not requesting one is not a
way to avoid one — required-review enforcement blocks the merge.

**Who reviews.** At least one `CODEOWNERS` owner. Two-person review required for platform and
SDK code, the wire contract, `conformance/`, CI workflow files, credentials handling,
Dockerfiles, and any deprecation (`standards.md` §14).

**What reviewers check, in this order.** Formatting and lint are *not* on this list — CI owns
them, and a reviewer raising them is out of scope (`styles.md` §0).

1. **Correctness.** Does it do what it claims? What happens on the failure path?
2. **Contract compatibility.** Read the diff for every JSON key and environment variable name.
   Renamed, retyped, or re-cased wire strings are the highest-cost defect class here
   (`standards.md` §5).
3. **Determinism.** Nondeterministic ordering, unseeded randomness, `now()`, float money,
   locale-dependent formatting, unstable output keys (`standards.md` §10.2).
4. **Tests.** Do they actually assert the behavior, or just execute it? Would this test fail if
   the change were reverted?
5. **Security.** Path traversal after substitution, unpinned deps, secrets, privilege
   escalation, injection into shell or SQL (`standards.md` §12).
6. **Scope.** Is anything here outside the stated purpose?
7. **Clarity.** Names, structure, comment accuracy.
8. **Cross-SDK parity.** For SDK changes: does the equivalent exist in the other three, and is
   a tracking issue filed?

**Author duties.** Respond to every comment. Push back with reasons — a review comment is advice,
not a work order, and "I did not do this because X" is a valid resolution. Do not silently
resolve. Force-pushing during review discards the reviewer's context; prefer adding commits and
letting the squash handle history.

**Reviewer duties.** Comment on the code, not the person. Ask questions rather than issuing
commands when intent is unclear. Approve when the change is a net improvement to the
codebase, not when it is perfect — a standard that requires perfection is a standard that
merges nothing.

**Time-box.** See §12.

---

## 10. Merge

- **Squash and merge** for pull requests into `main`. One PR → one commit → one changelog line.
- **Rebase and merge** only for a stacked series where each commit is independently meaningful
  and each has its own review.
- **Merge commits** are not used.
- The **author** merges, once the Definition of Done is met.
- **Never** merge with red required checks. **Never** use admin override except for a
  documented, time-boxed incident, recorded in `CC-DEVIATIONS.md` and followed up.
- **Never** force-push to `main`. Never push directly to `main`. Branch protection forbids it;
  do not seek to relax it.
- After merge: delete the branch, close the linked issue, and confirm release automation picked
  up the tag if one was intended.

---

## 11. Releases

Release only from a protected branch, only by CI, only with all checks green
(`standards.md` §11).

1. Confirm `CHANGELOG.md` has an entry for every user-visible change since the last tag.
2. Choose the version. `BREAKING CHANGE:` in any commit since the last tag → major. New
   backward-compatible capability → minor. Fixes only → patch.
3. Update the ecosystem version file: `pyproject.toml`, `build.gradle` version,
   `*.csproj` `PackageVersion`, `VERSION`/module tag for Go.
4. Tag `vMAJOR.MINOR.PATCH` (or the ecosystem's documented variant). **Tags are immutable —
   never move or reuse one.** `cc-dotnet-sdk` already warns about this; a moved tag silently
   breaks every pinned consumer and the reproducibility record.
5. Push the tag. CI builds, tests, scans, generates the SBOM, publishes the artifact, and
   creates the GitHub Release with notes derived from commit messages.
6. Verify the published artifact installs from a clean environment — the most common release
   failure is a package that builds but does not install.
7. For an SDK: record the `contracts/` version satisfied, and update the compatibility matrix
   (it is CI-generated; regenerate, do not hand-edit).
8. Announce breaking changes and deprecations to plugin owners **before**, not after, a major.

Ecosystem-specific publish paths, current state:

| SDK | Artifact | Destination |
| --- | --- | --- |
| `cc-py-sdk` | `cc-python-sdk` wheel/sdist | PyPI |
| `cc-dotnet-sdk` | `Usace.CC.Plugin` | GitHub Packages (`nuget.pkg.github.com/USACE/`) via `publish.yml` |
| `cc-java-sdk` | Gradle distribution (`gradle build -x test`, `gradle shadow`) | Maven-style repo |
| `cc-go-sdk` | Module | Go module proxy via tags — **currently pseudo-versioned only; needs real tags** |

**OPEN DECISION D-6** — which registries are authoritative for FFRD consumption.

---

## 12. Timelines

Response-time expectations, so that "waiting on review" is not open-ended. These are
commitments on maintainers, not on contributors.

| Situation | Target |
| --- | --- |
| Issue triage (label, assign or mark `needs-decision`) | 3 business days |
| First review on a ready PR | 3 business days |
| Re-review after author response | 2 business days |
| Two-person review completion | 5 business days |
| Security report acknowledgement (`SECURITY.md`) | 1 business day |
| Security fix or mitigation plan | 5 business days |
| Breaking-change / ADR notice period before merge | 10 business days |
| Deprecation window | 1 minor release and 6 months |
| Stale PR auto-flag (`stale`) | 30 days inactive; closes after 15 more unless objected |
| Stale issue auto-flag | 90 days; closed after 15 more |

Escalation: ping the `CODEOWNERS` handle in the PR after the target lapses. If still stalled
after a further 3 business days, raise it in the org discussion space and label
`needs-decision` — an unreviewed PR is a scheduling failure, and a repeatedly unreviewed repo
is a signal that it needs an owner or should be archived.

---

## 13. Closing a contribution

- **Accepted:** thank the contributor by name in the merge or release notes. Credit is not
  optional in a program that depends on a small number of contributors.
- **Declined:** say why, name the specific `standards.md` clause or design constraint, and say
  what would change the answer. Never leave a contribution with no disposition.
- **Deferred:** move to a named milestone with an owner, or close it. An open-ended "maybe
  later" issue is noise that hides the real backlog.
- Rejected external contributions are recorded in the ADR log if the reason is architectural,
  so the same proposal is not re-litigated next quarter.

---

## Appendix — copy-paste checklists

**Before opening a PR**
```
git fetch upstream && git rebase upstream/main
make ci
git log --oneline upstream/main..HEAD        # every message conventional? every commit buildable?
git diff --stat upstream/main...HEAD          # is anything here outside your stated scope?
grep -rniE '(AKIA|secret|password|token|BEGIN .*PRIVATE KEY)' $(git diff --name-only upstream/main...HEAD)
git status                                    # no .env, no local paths, no build output
```

**PR description minimum**
```
Closes #
What / Why
Wire contract: [none | keys added/changed: ... | BC for registered images: yes/no]
SDK parity: [n/a | go #__ java #__ dotnet #__]
Tests: [what was added; does it fail without the change?]
Deviations: [none | CC-DEVIATIONS.md entry added, expiry ____-__-__]
Docs/CHANGELOG updated: [yes | n/a]
```
