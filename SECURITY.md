# Security Policy

**Applies to:** all `USACE-Cloud-Compute` repositories.
**Implements:** `standards.md` §12.

## Reporting a vulnerability — do not open a public issue

Every repository in this organization is public. Filing a security issue publicly discloses
the vulnerability to everyone at the moment of filing, including people who would use it
against flood-risk infrastructure.

Report privately instead:

1. **GitHub private vulnerability reporting** — use the "Report a vulnerability" tab on the
   repository's Security page, where enabled.
2. **Email** — `cloudcompute-security@usace.army.mil` <!-- CONFIRM address before publishing -->
3. **Internal (USACE/FFRD staff):** the program's security channel per the FFRD SOP, plus a
   private note to the `CODEOWNERS` handles for the affected repository.

Include, as far as you safely can: affected repository and version, the component, a
reproduction or proof of concept, and the impact. **Do not include live credentials, real
PII/PHI/CUI, or production data in a report** — a report that leaks data to fix a leak has
traded one incident for another. Use synthetic evidence.

## What we consider reportable

Highest-value findings in this specific system, roughly in order:

- **Credentials or cloud roles reachable from a fork-triggered workflow.** This is the
  dominant risk in a public org with `allow_forking: true` and credential-bearing CI. See
  `standards.md` STD-SEC-001/002 and `CONTRIBUTING.md` §6.5.
- **Path traversal through payload substitution.** `{ENV::}` / `{ATTR::}` expansion writes to
  paths derived from data outside the plugin (STD-SEC-020).
- **A plugin or DAG reaching a DataSource it was not granted** (STD-PLG-005, STD-SEC-011).
- **Privilege escalation** via `privileged: true` or host-device mapping (STD-PLG-031).
- **Deserialization of untrusted bytes** — pickle, Java native serialization, unsafe YAML,
  `BinaryFormatter` (STD-SEC-022).
- **Unpinned native dependencies altering numerical output** — a GDAL/PROJ or solver skew
  changes FFRD results silently, without an error (STD-DEP-006/007). Treat a reproducible
  unexplained numeric change as a security-adjacent report.
- **PII/PHI/CUI present in any repository or its fork network** (STD-DATA-001/002).

## Response commitments

| Stage | Target |
| --- | --- |
| Acknowledge receipt | 1 business day |
| Triage and severity | 3 business days |
| Fix, mitigation, or published workaround | 5 business days for Critical/High |
| Advisory published, after fix is available | within 7 days of release |
| Credit to reporter | on request, in the advisory |

Severity follows CVSS v3.1, adjusted upward where a finding affects a system that produces
public-safety flood-risk outputs.

## If you find exposed credentials

Treat them as compromised immediately, whether or not you can use them:

1. Report privately, without including the secret value — name the file, commit, and line.
2. The owner rotates the credential.
3. Rotation happens **before** any public cleanup, because deleting a commit does not remove
   it from git object caches, forks, or GitHub's event stream.
4. The repository's history is scrubbed and the incident recorded.

## Hardening already required by standard

These are not optional in this org; a finding that one is missing is a bug:

- Least-privilege `permissions` in every workflow; third-party Actions pinned by SHA.
- No secrets in `pull_request` contexts; fork PRs require maintainer approval before running.
- OIDC-federated short-lived cloud roles in CI, not long-lived access keys.
- `gitleaks` on every PR with full history; image scanning at build **and** nightly.
- Non-root plugin containers; `privileged: false` by default.
- SBOM on every release artifact.

## Scope

In scope: every repository in `USACE-Cloud-Compute`, and the legacy `USACE` SDK repositories
listed as canonical in `standards.md` Appendix B.

Out of scope for this policy (route through normal channels instead): vulnerabilities in the
wrapped model executables themselves (HEC-HMS, HEC-RAS, Taudem), AWS platform issues, and
findings in a third-party plugin repository merely referenced from here.
