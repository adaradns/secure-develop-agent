# NestJS — Secure development guide

Complements [../owasp-top-10-2025.md](../owasp-top-10-2025.md). The generic *what/why* lives
there; this file is the *how* in NestJS. Each section maps to an OWASP A0X category.

## A01 — Broken Access Control
- Protect routes with `Guards`: `AuthGuard` for authentication, a `RolesGuard` + `@Roles()`
  decorator for authorization. A controller with no guard is a public endpoint — verify intent.
- For per-resource ownership (user can only read their own order), do the check in the
  service against the authenticated `request.user`, not only by role.
- Consider CASL (`@casl/ability`) for attribute-based rules instead of scattered `if role ===`.
- Default-deny: prefer a global guard + `@Public()` opt-out over per-route opt-in.

## A02 — Security Misconfiguration
- Add `helmet` and a strict CORS config (`app.enableCors({ origin: [...] })`) — never `origin: true`/`*` with credentials.
- Disable `x-powered-by`. Set `app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }))`.
- Never expose `/graphql` playground or Swagger in production without auth.

## A03 — Software Supply Chain
- `npm audit` / `pnpm audit` in CI; pin versions; review `postinstall` scripts of new deps.

## A04 — Cryptographic Failures
- Hash passwords with `bcrypt`/`argon2`, never MD5/SHA1. Keep secrets in env (`@nestjs/config`), never in code.

## A05 — Injection
- Validate every DTO with `class-validator` + `ValidationPipe` (`whitelist: true` strips unknown props).
- TypeORM/Prisma: use parameterized queries / query builder params — never string-concatenate
  user input into `query()`/`$queryRawUnsafe`.
- Shell: avoid `child_process.exec` with user input; use `execFile` with an args array.

## A07 — Authentication Failures
- Use `@nestjs/passport` + `passport-jwt`. Put JWTs in `httpOnly`, `Secure`, `SameSite` cookies —
  **not** `localStorage` (XSS theft). Set a short expiry + refresh-token rotation.
- Apply rate limiting on auth endpoints with `@nestjs/throttler`.

## A09 — Logging & Alerting
- Use a structured logger (pino/winston) and a redaction list so tokens/passwords never hit logs.

## A10 — Mishandling Exceptional Conditions
- Use a global `ExceptionFilter`; never leak stack traces or DB errors to the client in prod.
