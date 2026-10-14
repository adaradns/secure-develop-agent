---
name: secure-web
description: Secure web/application development based on OWASP Top 10:2025. Use when writing or reviewing backend/frontend code that touches endpoints, data access, secrets, auth, or external calls. Triggers on "security"/"seguridad", "OWASP", "injection"/"inyección", "auth"/"autenticación", "authorization"/"autorización", "secrets"/"secretos", "validation"/"validación", "rate limit", CORS, CSRF, or any web security review. Covers NestJS, Next.js, Spring Boot, Quarkus.
allowed-tools: Read, Grep, Glob
---

# Secure Web Development (OWASP Top 10:2025)

A **cross-cutting** skill for application code: endpoints/controllers, data access, secret
and environment-variable handling, and calls to external services. Read it before writing
or reviewing any of those.

## When it activates

Use it ALWAYS when writing or reviewing endpoints/controllers, data access, auth,
secret/env handling, or external calls. Also on mentions of "security", "OWASP",
"injection", "auth/authentication", "authorization", "secrets", "validation",
"rate limit", CORS, or CSRF. Triggers work in English and Spanish.

For LLM-specific risks (prompt injection, output handling, excessive agency) use the
**secure-llm** skill. For Dockerfiles, containers, Kubernetes or CI/CD use **secure-infra**.

## Instructions

1. **Detect the stack** from the project manifest before reading code:
   - `package.json` → Node. Look for `@nestjs/*` (NestJS) or `next` (Next.js).
   - `pom.xml` / `build.gradle` → Java. Look for `spring-boot` or `quarkus`.
2. **Load only what applies — never all references at once:**
   - The generic, language-agnostic checklist → [references/owasp-top-10-2025.md](references/owasp-top-10-2025.md)
   - The stack-specific "how to" → the matching file in `references/stacks/` (see below).
3. **Apply the checklist against the real code.** A grep match is not a finding — read the
   file in context first.
4. **Report findings** with severity (critical/high/medium/low) and a concrete fix anchored
   to `file:line`.
5. If the user asks for a written report, use `assets/secure-checklist-template.md` as the base.

## Hard rules (never negotiate)

- Never suggest logging secrets, tokens, or passwords in plain text.
- Never approve `localStorage` for session tokens without warning about the XSS risk.
- Never store sensitive data in plain text.
- Never grant admin permissions without prior review.
- Never skip a security review before a commit or a deployment.

## Additional resources

- Generic OWASP Top 10:2025 checklist: [references/owasp-top-10-2025.md](references/owasp-top-10-2025.md)
- Stack-specific guides (load the one that matches the project):
  - NestJS → [references/stacks/nestjs.md](references/stacks/nestjs.md)
  - Next.js → [references/stacks/nextjs.md](references/stacks/nextjs.md)
  - Spring Boot → [references/stacks/spring-boot.md](references/stacks/spring-boot.md)
  - Quarkus → [references/stacks/quarkus.md](references/stacks/quarkus.md)
- Report template: [assets/secure-checklist-template.md](assets/secure-checklist-template.md)
