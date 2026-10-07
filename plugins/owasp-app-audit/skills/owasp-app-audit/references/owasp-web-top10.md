# OWASP Top 10 (2021) — Classic Web/Application Risks (A01–A10)

Source: https://owasp.org/Top10/

This checklist audits **ordinary application code** for real, exploitable vulnerabilities. Each entry
gives the risk, a baseline severity, and concrete things to look for. Adjust severity to the actual
context (reachability by untrusted input, exposure of secrets, code-exec potential) per the main
SKILL.md guidance.

---

## A01 — Broken Access Control — Baseline: High
Users can act outside their intended permissions.

**Look for:**
- Endpoints/handlers with no authorization check, or checks only in the UI/client.
- IDOR: object IDs from the request used to fetch data without verifying ownership
  (`GET /orders/:id` returning any order).
- Missing role/permission checks on privileged actions; relying on hidden fields or `referer`.
- Path traversal (`../`) in file access; CORS configured with `*` plus credentials.
- Insecure direct access to admin routes; force-browsing not prevented.

## A02 — Cryptographic Failures — Baseline: High
Sensitive data exposed through weak or missing cryptography.

**Look for:**
- Secrets/keys/passwords hardcoded in source or committed config (`.env`, keys, tokens).
- Weak/broken algorithms: MD5, SHA1, DES, ECB mode, custom crypto.
- Passwords stored unhashed or with fast hashes (plain SHA) instead of bcrypt/scrypt/argon2.
- Sensitive data sent/stored without TLS; `http://` for credentials; disabled cert verification
  (`verify=False`, `rejectUnauthorized: false`).
- Hardcoded IVs/salts, predictable random (`Math.random`, `random` for tokens).

## A03 — Injection — Baseline: High
Untrusted input interpreted as code/commands (SQL, NoSQL, OS, LDAP) — includes XSS.

**Look for:**
- String-concatenated/interpolated SQL instead of parameterized queries/prepared statements.
- OS command exec built from input (`os.system`, `exec`, `subprocess` with `shell=True`, backticks).
- `eval`/`Function`/deserialization of untrusted input.
- XSS: unescaped user input rendered into HTML, `innerHTML`, `dangerouslySetInnerHTML`,
  template auto-escaping disabled.
- NoSQL/LDAP/XPath queries built from raw input.

## A04 — Insecure Design — Baseline: Medium
Missing or flawed security controls at the design level (not just a bug).

**Look for:**
- No rate limiting / anti-automation on auth, password reset, or expensive endpoints.
- Trust boundaries not enforced; security relying on client-side checks.
- Missing threat modeling for sensitive flows (payments, account recovery, privilege grants).
- Business-logic flaws (e.g. negative quantities, price tampering, workflow bypass).

## A05 — Security Misconfiguration — Baseline: High
Insecure default, incomplete, or ad-hoc configuration.

**Look for:**
- Debug mode on in production (`DEBUG=True`, stack traces returned to users).
- Default/sample credentials; admin consoles exposed.
- Missing security headers (CSP, HSTS, X-Content-Type-Options, X-Frame-Options).
- Overly permissive CORS; directory listing enabled; verbose error messages leaking internals.
- Unnecessary features/ports/services enabled; cloud storage buckets world-readable.

## A06 — Vulnerable and Outdated Components — Baseline: High
Using components with known vulnerabilities.

**Look for:**
- Dependencies pinned to old versions with known CVEs; unmaintained libraries.
- No dependency scanning / SCA; `npm audit` / `pip-audit` issues likely.
- Outdated framework/runtime versions; transitive deps not reviewed.
- Components pulled from untrusted sources.

## A07 — Identification and Authentication Failures — Baseline: High
Weaknesses in confirming user identity and managing sessions.

**Look for:**
- Weak password policy; no brute-force protection / lockout; credential stuffing possible.
- Session tokens that don't rotate on login, don't expire, or are exposed in URLs.
- Missing or weak MFA on sensitive accounts; predictable/guessable session IDs.
- JWT misuse: `alg: none` accepted, signature not verified, secret hardcoded/weak.
- Password reset tokens that are predictable or don't expire.

## A08 — Software and Data Integrity Failures — Baseline: High
Code and infrastructure that don't protect against integrity violations.

**Look for:**
- Deserialization of untrusted data (pickle, `unserialize`, Java/`ObjectInputStream`, YAML `load`).
- CI/CD pulling unverified code/plugins; auto-update without signature verification.
- Dependencies/scripts loaded from CDNs without integrity (SRI) checks.
- Unsigned artifacts or updates trusted implicitly.

## A09 — Security Logging and Monitoring Failures — Baseline: Medium
Insufficient logging/monitoring to detect and respond to breaches.

**Look for:**
- No logging of auth events, access-control failures, or high-value transactions.
- Logs missing context, or — conversely — logging secrets/PII in plaintext.
- No alerting/monitoring; logs only stored locally with no integrity protection.

## A10 — Server-Side Request Forgery (SSRF) — Baseline: High
The app fetches a remote resource using a user-supplied URL without validation.

**Look for:**
- Server-side `fetch`/`requests`/`curl`/`http.get` where the URL/host comes from user input.
- No allow-list of permitted hosts; internal addresses (`169.254.169.254`, `localhost`,
  RFC1918 ranges) reachable.
- Webhooks, URL previews, file-import-from-URL, PDF/image fetchers built from raw input.
- Redirects followed without re-validation.
