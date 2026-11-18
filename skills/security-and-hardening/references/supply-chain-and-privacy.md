# Supply-chain & Privacy Checklist

Two audits that don't show up in code review: the dependencies you didn't write, and the personal
data you're trusted with.

## Part 1 — Dependency / supply-chain audit

### Inventory

Use the project's own tooling; never guess versions.

| Ecosystem | Command                          |
| --------- | -------------------------------- |
| Node/npm  | `npm audit` / `npm audit --json` |
| Python    | `pip-audit`                      |
| Rust      | `cargo audit`                    |
| Go        | `govulncheck ./...`              |
| Any       | Dependabot / Renovate alerts     |

### Triage by reachability, then severity

A High in a transitive dependency you never call is not the same as a Critical in your request path.

1. **Is it reachable?** Does your code (or a dep you call) actually exercise the vulnerable function?
2. **What can it do?** RCE / auth bypass / data leak > DoS > info disclosure.
3. **Is there a fix?** Upgrade path, patch, or config mitigation available?
4. **Rank.** Reachable + high impact = act now. Unreachable + low impact = record and watch.

### Vetting a new package before adding it

Adding a dependency is a trust decision (`AGENTS.md`: never add a dependency without asking).

- **Maintenance:** recent commits/releases? one maintainer or several? bus factor?
- **Adoption:** download counts, dependents, age — new + popular is riskier (typosquat target).
- **Security history:** known CVEs? how fast are they patched?
- **Install scripts:** does it run `postinstall`/`preinstall`? what would it execute?
- **Surface:** could a smaller package or a built-in cover this? fewer deps is better.
- **Provenance:** published from the expected repo/CI? lockfile integrity respected?

### Remediation ladder

1. **Upgrade** to a fixed version (prefer).
2. **Patch** (override/pin a fixed transitive version).
3. **Replace** the package with a safer/fewer-dep alternative.
4. **Mitigate by config** (disable the vulnerable feature/code path).
5. **Accept** — only with a written reason, an owner, and a revisit date.

An accepted High is a debt: it must be logged, not silently dropped.

## Part 2 — Privacy & data protection

### Classify

- **Personal data:** anything identifying a person (name, email, IP, device id, location).
- **Sensitive / special category:** health, biometrics, financial, children's data,
  credentials — GDPR Art. 9 territory; higher bar.
- **Regulated:** GDPR (EU), CCPA/CPRA (California), or sector rules (HIPAA, PCI-DSS).

### Data map — for each field, answer

- **Collected** where, and is it _necessary_? (minimization)
- **Stored** where, encrypted at rest, for how long? (retention limit)
- **Transmitted** to whom, over TLS, to which third parties?
- **Logged** — does it leak into logs, traces, error messages, or analytics?
- **Deleted** — can you honor a delete/export/access request?

### Minimization rules

- Collect only what the feature needs; drop the rest.
- Never log PII, tokens, or credentials — redact at the boundary (see A09).
- Use **synthetic** data in dev/test; copies of real PII are still real PII.
- Encrypt PII at rest and in transit; keep keys out of the repo.

### Third parties

- What personal data leaves the boundary, to whom?
- Is there a DPA / processor agreement?
- Does consent (if relied on) cover this specific use?
- Can you delete data held by the processor?

### Rights to honor (GDPR/CCPA)

- **Access / export** — produce a user's data on request.
- **Erasure** — delete across all stores, backups, and processors.
- **Correction**, **portability**, and (CCPA) **opt-out of sale/sharing**.

### Output for this part

- A data map (field → where → why → retention).
- Findings: over-collection, unencrypted PII, PII in logs, unaccounted third parties.
- Each finding gets a severity and a fix; Critical/High block, like app-sec.
