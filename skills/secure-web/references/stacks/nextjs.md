# Next.js — Secure development guide

Complements [../owasp-top-10-2025.md](../owasp-top-10-2025.md). Covers App Router (Server
Components, Server Actions, Route Handlers) and the client boundary. Each section maps to OWASP A0X.

## A01 — Broken Access Control
- **Re-check auth inside every Server Action and Route Handler** — middleware alone is not
  enough (actions are public POST endpoints). Validate `session`/`user` at the top of each.
- Never rely on hiding a UI control for authorization; the server action must enforce it.
- Beware the `'use server'` boundary: anything exported is a callable endpoint.

## A02 — Security Misconfiguration
- Set security headers (CSP, `X-Frame-Options`, HSTS) via `next.config.js` `headers()` or middleware.
- Only `NEXT_PUBLIC_*` env vars reach the browser — double-check no secret is prefixed that way.
- Don't import server-only modules into Client Components; use the `server-only` package to fail fast.

## A04 — Cryptographic Failures
- Keep API keys and DB URLs server-side only (no `NEXT_PUBLIC_`). Access secrets in Server
  Components / Route Handlers, never pass them as props to Client Components.

## A05 — Injection / XSS
- Avoid `dangerouslySetInnerHTML`; if unavoidable, sanitize with DOMPurify.
- Validate all input in Server Actions/Route Handlers with `zod` before use — the client form
  validation is cosmetic.
- Parameterize DB queries (Prisma/Drizzle); never concatenate input into raw SQL.

## A07 — Authentication Failures
- Use a vetted library (Auth.js/NextAuth, Clerk, Lucia). Store sessions in `httpOnly`,
  `Secure`, `SameSite=Lax/Strict` cookies — **not** `localStorage`.
- CSRF: Server Actions have built-in origin checks, but custom Route Handlers that mutate
  state need explicit CSRF protection or a strict `SameSite` cookie + origin check.

## A08 — Data Integrity
- Pin dependency versions; be wary of untrusted data flowing into `eval`/dynamic `import()`.

## A10 — Mishandling Exceptional Conditions
- Use `error.tsx` boundaries; never surface raw error messages/stack traces to users in prod
  (`reactProductionProfiling` off, generic error pages).
