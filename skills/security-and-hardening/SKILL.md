---
name: security-and-hardening
description: "Use this when a change or milestone warrants a security pass: untrusted input, an auth/session/data path, a new dependency, or personal data. The goal is to harden the code (OWASP Top Ten), audit dependencies (npm audit / supply chain), and check privacy (GDPR/CCPA), reporting findings by severity. You are not allowed to ship a Critical/High finding without fixing it or a written, dated acceptance. A standalone skill run on demand, not a workflow stage. Trigger phrases: security audit, harden this, OWASP, threat model, dependency audit, supply chain, GDPR."
argument-hint: "Point at the target, e.g. the login flow, src/api handler, or 'dependencies'."
---

# Security and Hardening

Security is not a phase you bolt on at the end — it's a property you build in and then **prove**. This skill hardens code against the ways it actually gets broken, and audits the two surfaces that are easy to forget: the **dependencies** you didn't write and the **personal data** you're trusted with.

It is a **standalone skill**, not a workflow phase and not an automatic stage. You apply it when the work calls for it: hardening a handler, auditing a login flow, triaging a dependency audit, or checking a feature that accepts untrusted data. If you're inside `fight`, its findings flow back through the normal loop (fixes are built test-first and re-verified); standalone, you still fix and prove.

This is defensive work: finding and closing vulnerabilities, hardening, and auditing. It never helps build an exploit or attack a system you don't own.

## The Model

| Thing               | What it is                                                                       | Where it lives                                                     |
| ------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| **Asset**           | What's worth protecting: credentials, PII, money, integrity, uptime.             | named in the threat model                                          |
| **Trust boundary**  | Where untrusted data enters a trusted zone. Crossing it is evil until validated. | the interfaces in `docs/architecture/architecture.md`              |
| **Untrusted input** | Anything a user, client, third party, or another service supplies.               | request bodies, params, headers, files, webhooks, queues           |
| **Finding**         | One weakness, with severity, location, and the fix.                              | the review/report                                                  |
| **OWASP Top Ten**   | The checklist for app-sec coverage (2021).                                       | [references/appsec-checklist.md](./references/appsec-checklist.md) |

**The one rule under all of them:** treat every input as hostile, every secret as sacred, and every dependency as a liability until it earns its place.

## When to Use

- A feature accepts **untrusted data** (input, uploads, webhooks, third-party payloads).
- It manages **auth, sessions, cookies, or credentials** — or you're checking a login flow is safe.
- It **stores, transmits, or logs** personal data, or a privacy regime (GDPR/CCPA) applies.
- You integrated or are vetting a **dependency**, or a package-manager audit reported findings.
- You want a **threat model** for a design, or a security review of a diff.

**Do NOT use** for: non-security code review (that's the loop's `review`), general correctness or tests
(the loop's `implement`/`verify`), or deciding _what_ to build (`brainstorming`). This skill hardens and
audits on demand; it doesn't replace the loop's review or verification.

**Standalone by design.** No orchestrator dispatches it and no phase depends on it. It's a tool you pick up when a change or a milestone warrants a security, dependency, or privacy pass.

## Core Rules

- **Assume breach.** Design so one failure doesn't become the whole system. Fail closed, default deny.
- **Validate at the boundary, once.** Parse untrusted input into a known-good shape at the edge, then trust it internally. Never re-validate the same value in ten places — and never skip the edge.
- **Never hand-roll the hard parts.** Use vetted libraries for auth, crypto, password hashing, and token signing. Homegrown crypto is a finding, not cleverness.
- **Secrets never touch the repo.** No keys, tokens, or passwords in code, tests, logs, or `.env` commits. Per `AGENTS.md`, `.env` is never edited; `.env.example` holds placeholders.
- **Least privilege, everywhere.** DB users, API scopes, and tokens get the minimum they need. No wildcard CORS, no `SELECT *` into a response, no admin-by-default.
- **Every finding is actionable.** Severity, exact location (file:line), what's wrong, why it matters, and the fix — or it isn't a finding.
- **Severity drives the gate.** Critical and High **block** the change (back to `implement`); Medium/Low are recorded and scheduled. Triage is not a vibe; see the references for the bar.
- **Audit the supply chain like your own code.** A new dependency is a trust decision. Uncapped, unmaintained, or typosquatted packages are findings.
- **Personal data is a debt.** Collect the minimum, keep it only as long as needed, encrypt it, and know where every copy lives.
- **Prove the fix.** A hardening change isn't done until `verify` shows a **regression test that fails without it** — a test that would have caught the bug.

## Procedure

Pick the mode the work needs. They compose — a login flow touches mode 1 and often mode 3.

### Mode 1 — App-sec hardening (OWASP)

1. **Map the trust boundaries.** List every place untrusted data enters, and every asset behind it. Walk the interfaces in `docs/architecture/architecture.md`.
2. **Threat-model the paths.** For each boundary, ask: how could this be abused (spoof, tamper, inject, escalate, exfiltrate, deny)? Use STRIDE as a lens; keep it proportional to the tier.
3. **Check against the Top Ten.** Walk [references/appsec-checklist.md](./references/appsec-checklist.md) — injection, broken access control, auth failures, crypto failures, misconfiguration, integrity, logging, SSRF, insecure design.
4. **Harden in place.** Validate/encode at the boundary, parameterize queries, enforce authz server-side, hash passwords with a modern KDF (argon2/bcrypt/scrypt), set cookie flags (`HttpOnly`, `Secure`, `SameSite`), lock down CORS and headers.
5. **Write the regression test first** for each fix — the one that reproduces the weakness.

### Mode 2 — Dependency / supply-chain audit

1. **Inventory.** Use the project's own tooling (`npm audit`, `pip-audit`, `cargo audit`, Dependabot) — don't guess versions.
2. **Triage by reachability + severity.** A High in a transitive dep you never call is not the same as a Critical in your request path. Rank by _what's actually reachable_ and _what it can do_.
3. **Assess new/unknown packages** before adding: maintenance, release recency, download counts, known CVEs, install scripts, and whether a smaller/built-in alternative exists. See [references/supply-chain-and-privacy.md](./references/supply-chain-and-privacy.md).
4. **Remediate:** upgrade, patch, pin, replace, or mitigate-by-config. Never add a dependency to fix this without asking (`AGENTS.md`).
5. **Record** what was accepted and why — an unpatched High that's accepted needs a written reason and a revisit date.

### Mode 3 — Privacy / data protection

1. **Classify the data.** What's personal, sensitive, or regulated (GDPR/CCPA)? Where is it collected, stored, transmitted, logged, and shared?
2. **Minimize.** Drop fields you don't need; don't log PII; redact in errors and telemetry.
3. **Protect in transit and at rest** (TLS, encryption, key management), and honor the data-subject rights the regime requires (access, delete, export).
4. **Check third parties.** What leaves the boundary, to whom, under what agreement?
5. Walk [references/supply-chain-and-privacy.md](./references/supply-chain-and-privacy.md) for the checklist.

### Then — report and gate

Report findings by severity (Critical / High / Medium / Low), each with location and fix. Critical/High block and go back to `implement`. Record the outcome in the feature's `specs/NNN-slug/status.md` notes (or the review when standalone). If you're inside fight mode, the hardened change then goes through `review` and `verify` like any other work.

## Red Flags

| Thought                                          | Reality                                                                                         |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| "It's an internal service, it doesn't need auth" | "Internal" is a network position, not a trust boundary. Attackers get inside.                   |
| "I'll sanitize it later, it's fine for now"      | Injection is written at the boundary or not at all. Later never comes.                          |
| "I'll write my own token/crypto to control it"   | Home-rolled crypto is a finding. Use the vetted library.                                        |
| "The framework handles security for me"          | It handles _some_ — authz, misconfiguration, and SSRF are still yours.                          |
| "The dependency is popular, so it's safe"        | Popular packages get compromised. Check CVEs, scripts, and maintenance.                         |
| "This finding is High but we'll ship anyway"     | High blocks the change, or gets a written acceptance with a revisit date.                       |
| "Logging the whole request will help debugging"  | That's how secrets and PII end up in logs. Redact at the boundary.                              |
| "It's just test data, privacy doesn't apply"     | Test copies of real PII are real PII. Use synthetic data.                                       |
| "The audit reported it, so I'll bump everything" | Triage by reachability and severity — blind upgrades break things and don't fix reachable risk. |

## Completion Check

Security work is done when ALL are true:

- [ ] Every trust boundary is identified and every untrusted input is validated once, at the edge.
- [ ] The OWASP Top Ten was walked for the touched surfaces, with each item marked pass / finding / N/A.
- [ ] Auth, sessions, and cookies use vetted mechanisms; no home-rolled crypto; secrets are out of the repo.
- [ ] Findings are reported by severity with location and fix; Critical/High block and went back to `implement`.
- [ ] Dependencies were audited with the project's tooling; each finding is fixed, mitigated, or accepted in writing.
- [ ] Personal data is minimized, encrypted, and not logged; third parties receiving it are accounted for.
- [ ] Each hardening fix has a regression test that fails without it; `verify` is green.
- [ ] The outcome is recorded in the feature's `specs/NNN-slug/status.md` notes (or the review when standalone).

If any box is unchecked, the hardening isn't finished — do not ship it.
