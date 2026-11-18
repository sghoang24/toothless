# App-sec Checklist (OWASP Top Ten)

Walk this for the surfaces the change touches. Mark each item **pass** / **finding** / **N/A** — an
unwalked item is not a pass. The OWASP Top Ten (2021) is the spine; the notes are what to actually
look for in code.

## How to use

For each item: find where it applies in the diff or design, test the claim, and record the result.
A finding gets a severity, a location (`file:line`), and a fix. Critical/High block the change.

## A01 — Broken Access Control

The most common serious flaw. Authz must be enforced **server-side**, per request, on every path.

- Every endpoint checks _both_ authentication (who) and authorization (what they may do).
- No "hidden" admin routes, no client-side-only checks, no sequential IDs exposed without an
  ownership check (IDOR).
- Default deny: a new route is inaccessible until explicitly allowed.
- Object-level checks: the caller owns / may access _this_ record, not just "is logged in".
- Deny by default on CORS — no wildcard `*` with credentials.

## A02 — Cryptographic Failures

Weak or missing crypto; secrets in the open.

- TLS everywhere data crosses the wire; no fallback to plaintext.
- Passwords hashed with a modern KDF (argon2id / bcrypt / scrypt) + per-user salt; never plain or
  weak (MD5/SHA1) hashes.
- Secrets from a secret store / env, never in code, tests, logs, or `.env` commits.
- Strong, vetted algorithms (AES-GCM, ChaCha20-Poly1305); no ECB, no home-rolled crypto.
- Sensitive data encrypted at rest (PII, tokens, backups).

## A03 — Injection

Untrusted data reaching an interpreter: SQL, NoSQL, OS shell, LDAP, templates, XML.

- Queries are **parameterized** — string concatenation of input into SQL/NoSQL is a finding.
- No `eval`/`exec` on input; no shell built from user strings (use arg arrays).
- Output encoded for its context (HTML, attribute, JS, URL) — XSS is injection into a browser.
- Template engines auto-escape; `|safe`/raw render on user data is a finding.
- SSRF: user-supplied URLs are validated against an allowlist; no fetching arbitrary hosts.

## A04 — Insecure Design

Missing or ineffective controls by design — not an implementation bug.

- Threat model exists for the risky paths (STRIDE); trust boundaries are explicit.
- Rate limiting / anti-automation on auth, signup, and expensive operations.
- Business-logic limits (e.g. order quantity, transfer amounts) enforced server-side.
- Fail closed: an error path denies access rather than granting it.

## A05 — Security Misconfiguration

Defaults left unsafe.

- Debug/verbose errors off in production; no stack traces to clients.
- Security headers set (CSP, HSTS, `X-Content-Type-Options`, frame options).
- Cookies: `HttpOnly`, `Secure`, `SameSite=Lax/Strict`.
- No default/example credentials; unused features and ports disabled.

## A06 — Vulnerable & Outdated Components

See Mode 2 (dependency audit). Here: are known-vulnerable versions in use, and is there a process to
find out?

- `npm audit` / `pip-audit` / `cargo audit` run and triaged.
- No unpinned or abandoned critical deps in the request path.

## A07 — Identification & Authentication Failures

- Credential stuffing / brute force resisted (rate limit, lockout, or MFA).
- Session IDs are strong, random, rotated on login, and invalidated on logout.
- Password policy sane; no password in URLs or logs; secure reset flow (single-use, expiring token).
- Session cookies can't be fixed by an attacker (no session fixation).

## A08 — Software & Data Integrity Failures

- Deserialization of untrusted data avoided or strictly typed (no native `pickle`/Java serialization
  on input).
- Updates and artifacts verified (signatures / checksums / lockfiles).
- CI/CD: no unsigned or unverified artifact promoted; secrets not exposed to untrusted PRs.

## A09 — Security Logging & Monitoring Failures

- Auth events, access-control failures, and input-validation failures are logged.
- Logs **never** contain secrets, tokens, passwords, or raw PII.
- Logs are tamper-resistant and actually reviewed / alerted on.

## A10 — Server-Side Request Forgery (SSRF)

- Server-initiated requests to user-influenced targets are allowlisted.
- Cloud metadata endpoints (169.254.169.254) and internal ranges are blocked.
- Redirects on server-side fetches are not blindly followed.

## Severity bar

| Severity     | Test                                                            | Action                        |
| ------------ | --------------------------------------------------------------- | ----------------------------- |
| **Critical** | Direct path to takeover, data loss, or RCE with untrusted input | Blocks; back to `implement`   |
| **High**     | Exploitable serious weakness (authz bypass, SQLi, secret leak)  | Blocks; back to `implement`   |
| **Medium**   | Real weakness needing a precondition or causing limited harm    | Record and schedule           |
| **Low**      | Hardening / defense-in-depth with no direct exploit             | Record; fix opportunistically |
