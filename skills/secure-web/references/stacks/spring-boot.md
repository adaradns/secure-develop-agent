# Spring Boot — Secure development guide

Complements [../owasp-top-10-2025.md](../owasp-top-10-2025.md). Assumes Spring Security on the
classpath. Each section maps to an OWASP A0X category.

## A01 — Broken Access Control
- Define a `SecurityFilterChain` with explicit `authorizeHttpRequests` — default-deny
  (`anyRequest().authenticated()`), then open specific routes. Avoid `permitAll()` catch-alls.
- Method security: `@EnableMethodSecurity` + `@PreAuthorize("hasRole('ADMIN')")` /
  `@PreAuthorize("#id == authentication.principal.id")` for per-resource ownership.
- Never disable security for "convenience" in a profile that can reach production.

## A02 — Security Misconfiguration
- Lock down CORS via `CorsConfigurationSource` — explicit origins, never `*` with credentials.
- Don't expose all Actuator endpoints: `management.endpoints.web.exposure.include` only what's
  needed, and secure `/actuator` behind auth. Disable `/actuator/env`, `/heapdump` in prod.
- Keep `server.error.include-stacktrace=never` and `include-message=never` in prod.

## A03 — Software Supply Chain
- Use the OWASP Dependency-Check or `mvn versions`/Gradle plugin in CI; pin versions.

## A04 — Cryptographic Failures
- Password hashing via `BCryptPasswordEncoder`/`Argon2PasswordEncoder` (DelegatingPasswordEncoder).
- Externalize secrets (env, Vault, config server) — never commit `application.properties` secrets.

## A05 — Injection
- JPA/Hibernate: use bound parameters (`:param`, `?1`) — never concatenate input into JPQL/SQL.
  For native queries use `@Param`, and avoid building `WHERE` clauses by string concat.
- Validate request bodies with `@Valid` + Bean Validation (`@NotBlank`, `@Size`, `@Pattern`).

## A07 — Authentication Failures
- For token auth use Spring Security OAuth2 Resource Server (JWT) with signature + expiry
  validation. For session auth, keep `JSESSIONID` cookie `HttpOnly`, `Secure`, `SameSite`.
- Enable CSRF protection for session/cookie flows (on by default — don't blindly `.csrf().disable()`).
- Rate-limit auth endpoints (Bucket4j or gateway-level).

## A09 — Logging & Alerting
- Use SLF4J + Logback with a masking converter so credentials/tokens are never logged.

## A10 — Mishandling Exceptional Conditions
- Centralize with `@ControllerAdvice` / `@ExceptionHandler`; return sanitized error responses,
  never raw exceptions or SQL error text.
