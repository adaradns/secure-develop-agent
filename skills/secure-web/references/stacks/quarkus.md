# Quarkus — Secure development guide

Complements [../owasp-top-10-2025.md](../owasp-top-10-2025.md). Assumes `quarkus-security` and
typically RESTEasy Reactive + Hibernate ORM/Panache. Each section maps to an OWASP A0X category.

## A01 — Broken Access Control
- Annotate endpoints with `@RolesAllowed`, `@Authenticated`, or `@PermitAll` explicitly.
  Enable `quarkus.security.deny-unannotated-members=true` so an un-annotated endpoint is denied,
  not silently public.
- For per-resource ownership, check the resource owner against the injected `SecurityIdentity`
  in the service layer.

## A02 — Security Misconfiguration
- CORS: configure `quarkus.http.cors.origins` with explicit origins, never `/.*/` with credentials.
- Don't expose dev UI / OpenAPI / health in prod without auth (`quarkus.swagger-ui.always-include=false`).
- Keep `quarkus.log.console` free of sensitive config; disable detailed error pages in prod.

## A03 — Software Supply Chain
- Run dependency scanning in CI; pin the Quarkus BOM version; review extensions you add.

## A04 — Cryptographic Failures
- Hash passwords with Argon2/BCrypt (not plain Elytron MD5 realms). Keep secrets in env or Vault
  (`quarkus-vault`), never in `application.properties` committed to git.

## A05 — Injection
- Panache/Hibernate: use named/positional parameters (`Panache.find("name = ?1", name)`) — never
  concatenate user input into the query string.
- Validate inputs with Hibernate Validator (`@Valid`, `@NotBlank`, `@Pattern`) on REST resources.

## A07 — Authentication Failures
- Prefer `quarkus-oidc` for OAuth2/OIDC token auth (signature + expiry validated by the extension).
  For JWT use `quarkus-smallrye-jwt` with issuer/expiry checks.
- Cookies: `HttpOnly`, `Secure`, `SameSite`. Rate-limit auth endpoints at the gateway.

## A09 — Logging & Alerting
- Use JBoss Logging with a formatter that masks tokens/passwords; log auth failures for alerting.

## A10 — Mishandling Exceptional Conditions
- Map exceptions with `ExceptionMapper<T>`; return clean error payloads, never raw stack traces
  (`quarkus.http.body... ` / disable detailed exceptions in prod).
