# OWASP Top 10:2025 — Generic checklist

Reference for auditing/reviewing security in any project (backend Java/Quarkus/Spring Boot,
Node/NestJS, frontend Next.js, ReactJS, containers). Format per category: **What it is** ·
**How it shows up** · **Checklist**.

Do not assume the project already has X or Y implemented: verify each point against the
real code before reporting it as resolved or pending.

## Index
- [A01 — Broken Access Control](#a01)
- [A02 — Security Misconfiguration](#a02)
- [A03 — Software Supply Chain Failures](#a03)
- [A04 — Cryptographic Failures](#a04)
- [A05 — Injection](#a05)
- [A06 — Insecure Design](#a06)
- [A07 — Authentication Failures](#a07)
- [A08 — Software or Data Integrity Failures](#a08)
- [A09 — Security Logging and Alerting Failures](#a09)
- [A10 — Mishandling of Exceptional Conditions](#a10)

---

<a id="a01"></a>
## A01:2025 — Broken Access Control
**What it is:** a user accesses resources or actions they should not (endpoints, another
user's records, admin actions without being an admin).

**How it shows up:**
- Endpoints with no authentication guard/middleware.
- Authorization checked only in the frontend, never in the backend.
- Predictable resource IDs without ownership checks (`GET /orders/123` without checking
  that `123` belongs to the authenticated user).

**Checklist:**
- [ ] Every endpoint that exposes data or triggers actions has an authentication guard
  (JWT, API key, session) — never rely on "nobody will guess the URL".
- [ ] Authorization is verified in the backend, not just by hiding a button in the UI.
- [ ] Resource ownership is validated (the authenticated user owns/has permission over the
  record they request), not just that they are logged in.
- [ ] Roles/permissions are verified with a declarative framework mechanism
  (`@RolesAllowed` in Quarkus, guards in NestJS, middleware in Next.js) instead of manual
  scattered checks.

---

<a id="a02"></a>
## A02:2025 — Security Misconfiguration
**What it is:** insecure default configuration, missing security headers, over-exposed
features, poorly separated environments.

**How it shows up:**
- Missing rate limiting on expensive endpoints (external API/LLM calls, email sending,
  bulk write operations).
- No security headers (`helmet` in Node, equivalents in other stacks).
- Open CORS (`*`) without a real need for it.
- Payload validation that does not reject undeclared fields.
- Stack traces or internal details leaked in production responses.

**Checklist:**
- [ ] Rate limiting applied consistently on ALL expensive endpoints, not just some —
  review case by case, do not assume a global guard covers everything.
- [ ] Security headers configured (`helmet` or equivalent) and CORS with explicit origins,
  not a wildcard.
- [ ] Input validation with a field whitelist (reject fields not declared in the
  DTO/schema), not just types.
- [ ] The production `NODE_ENV`/profile does not expose stack traces, debug endpoints, or
  internal documentation (Swagger without auth, for example).

---

<a id="a03"></a>
## A03:2025 — Software Supply Chain Failures
**What it is:** risk from compromised, outdated, or vulnerable third-party dependencies,
packages, or binaries — including package-manager libs and external binaries invoked from
the code.

**How it shows up:**
- Known vulnerabilities reported by the package manager's auditor (`npm audit`,
  `mvn dependency-check`, etc.) left unresolved.
- Wide version ranges on sensitive dependencies (auth, crypto, parsing).
- External binaries (CLIs, system tools) invoked without pinning a version or validating
  their provenance.

**Checklist:**
- [ ] Run the stack's dependency auditor after installing or updating, and resolve
  high/critical issues.
- [ ] Lockfile committed and used in CI with a reproducible install (`npm ci`, `mvn -o`,
  etc.), not a "free" install.
- [ ] Any external binary executed from the code: pinned version, installed from a trusted
  source, and its output treated as untrusted input (see A05).
- [ ] Review existing version overrides/pins when new CVEs appear.

---

<a id="a04"></a>
## A04:2025 — Cryptographic Failures
**What it is:** sensitive data unencrypted in transit or at rest, poorly managed secrets,
homegrown or misapplied cryptography.

**How it shows up:**
- API keys, tokens, or credentials in a committed `.env` or in source code.
- Outbound traffic to external services over HTTP instead of HTTPS.
- Password hashing with weak algorithms or without salt.

**Checklist:**
- [ ] Secret files (`.env` and similar) are in `.gitignore`; a `.env.example` with no real
  values is provided.
- [ ] All outbound traffic to external services is HTTPS.
- [ ] Key rotation plan if a leak is suspected; secrets never appear in logs (see A09).
- [ ] Hashing/encryption uses standard ecosystem libraries (never a homegrown crypto
  implementation).

---

<a id="a05"></a>
## A05:2025 — Injection
**What it is:** untrusted input interpreted as a command or query (SQL, OS commands,
NoSQL, LDAP, etc.).

**How it shows up:**
- Queries built by string concatenation instead of parameterized.
- System commands executed with an interpolated string (`exec`/`execSync` with template
  strings) instead of an argument array.
- Input values (IDs, slugs, parameters) interpolated directly into URLs or commands
  without validating their format.

**Checklist:**
- [ ] All database queries use parameterization or the ORM/client's query builder — zero
  string concatenation with user input.
- [ ] System command execution uses the variant that takes arguments as an array
  (`execFile`, not `exec` with a string), and never goes through an interpolated shell.
- [ ] Any value interpolated into a URL, command, or query (IDs, slugs) is validated
  against an expected format (regex/whitelist) before use.

---

<a id="a06"></a>
## A06:2025 — Insecure Design
**What it is:** design flaws — security controls missing from the initial design of a
feature, not one-off implementation bugs.

**How it shows up:**
- Expensive operations (paid API calls, LLM, bulk sending) without usage limits or
  per-user cost control.
- No threat modeling before exposing a new endpoint.
- No idempotency mechanisms on operations that should not be repeated.

**Checklist:**
- [ ] Every feature that triggers an expensive operation defines usage/cost limits per user
  from the design stage, not as a patch afterward.
- [ ] Before exposing a new endpoint: who can call it? what resources does it consume? what
  happens if it is called many times in a row?
- [ ] Non-idempotent operations have a control mechanism (freshness window, locks,
  idempotency keys) to avoid expensive reprocessing or duplicates.

---

<a id="a07"></a>
## A07:2025 — Authentication Failures
**What it is:** weak or absent authentication, poor session management.

**How it shows up:**
- Tokens without expiration or with excessively long expiration.
- Authentication scheme hand-rolled instead of using a proven library/standard (JWT with
  standard libraries, OAuth, etc.).
- No rate limiting on login/registration endpoints.

**Checklist:**
- [ ] Session tokens have reasonable expiration and are validated server-side on every
  request.
- [ ] No authentication secret (signing keys, credentials) lives on the client.
- [ ] Specific rate limiting on login/password-recovery endpoints to slow down brute
  force.
- [ ] A proven library/standard is used for JWT/OAuth/sessions — never a homegrown scheme.

---

<a id="a08"></a>
## A08:2025 — Software or Data Integrity Failures
**What it is:** trusting data, updates, or dependencies without verifying their integrity.

**How it shows up:**
- CI that does not use a reproducible install from the lockfile.
- Data generated by external systems (including an LLM) persisted without validating
  ranges, types, or expected enums.
- Deserialization of third-party content without prior validation.

**Checklist:**
- [ ] CI/CD uses a reproducible install from the lockfile, not a "free" install.
- [ ] Any data generated by an external system or an LLM is validated (ranges, types,
  enums) before being persisted — do not trust that "the LLM always returns the requested
  format".
- [ ] Integrity constraints also exist at the database level (`CHECK`, constraints), not
  just in the application code.
- [ ] Third-party content is not executed or deserialized without validating its structure
  first.

---

<a id="a09"></a>
## A09:2025 — Security Logging and Alerting Failures
**What it is:** lack of logs useful for investigating incidents, or logs that leak
sensitive data, or absence of alerts on repeated failures.

**How it shows up:**
- Debug-level logs that include sensitive content (full payloads, LLM responses, PII) also
  in production.
- Use of stray `console.log`/prints instead of the framework's structured logger.
- No logging of security events (auth failures, rate limit exceeded).

**Checklist:**
- [ ] Secrets, tokens, PII, or full sensitive content are never logged — debug logs with
  that content are limited to development, never in production.
- [ ] The framework's structured logger is used, not stray prints.
- [ ] Security events (auth failures, rate limit exceeded, repeated validation errors) are
  recorded with enough context to investigate.
- [ ] There is some alerting mechanism for repeated or anomalous failures, not just passive
  logs nobody reviews.

---

<a id="a10"></a>
## A10:2025 — Mishandling of Exceptional Conditions
**What it is:** incorrect handling of errors or anomalous conditions: failing open,
swallowing errors, or leaking internal details in error messages.

**How it shows up:**
- A query/operation failure is returned as an empty result (`[]`, `null`) instead of being
  propagated as an error, producing false negatives (e.g. a 404 when the database
  connection actually failed).
- Internal exceptions exposed to the client with a stack trace or internal message.
- Security checks that, on an unexpected error, let the operation through instead of
  denying it.

**Checklist:**
- [ ] Explicitly distinguish "no data" (a legitimate empty result) from "the operation
  failed" (must throw/propagate the error) — do not swallow errors as if they were empty
  results.
- [ ] On a failure in a security check, the default is to deny (fail closed), never to let
  it through.
- [ ] Error messages to the client are controlled and generic (use framework exceptions:
  `NotFoundException`, `BadRequestException`, etc.); the full detail stays only in internal
  logs.

---

## How to use this reference when reviewing a project

1. Do not assume the state of anything — review the real code against each checklist.
2. Prioritize by severity: Broken Access Control, Injection, and Authentication Failures
   are usually directly exploitable; Logging and Insecure Design are more structural but
   equally important in the medium term.
3. When reporting a finding, state: OWASP category, severity (critical/high/medium/low),
   file/line if applicable, and the concrete fix — not just "this is insecure".
