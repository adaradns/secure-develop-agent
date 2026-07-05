---
name: secure-develop
description: Secure development skill based on OWASP Top 10:2025 and OWASP Top 10 for LLM Applications 2025. Use when writing or reviewing code that touches user input, data, secrets, or LLM integration. Triggers on "security"/"seguridad", "OWASP", "injection"/"inyección", "auth"/"autenticación", "authorization"/"autorización", "secrets"/"secretos", "prompt injection", "validation"/"validación", "rate limit", or any security review request.
allowed-tools: Read, Grep, Glob
---

# Secure Development

A **cross-cutting** skill: it applies to every module, not just one. Read it before writing
or reviewing code that touches user input, data, secrets, or the LLM.

## When it activates

Use it ALWAYS when writing or reviewing: endpoints/controllers, data access, secret and
environment-variable handling, calls to external services. Also activate on mentions of
"security", "OWASP", "injection", "auth/authentication", "authorization", "secrets",
"prompt injection", "validation", "rate limit", or any request for a security review.

## Instructions

1. Identify the domain of the request (endpoints, data access, secrets, environment
   variables, security mentions, OWASP, auth, LLM, prompt injection, rate limiting) and
   read ONLY the matching reference — do not load them all at once.
2. Apply that reference's checklist against the real code.
3. Report findings with severity (critical/high/medium/low) and a concrete fix.
4. If the user asks for a report, use `assets/secure-checklist-template.md` as the base.

## Hard rules (never negotiate)

- Never suggest logging secrets, tokens, or passwords in plain text.
- Never approve `localStorage` for session tokens without warning about the XSS risk.
- Never store sensitive data in plain text.
- Never grant admin permissions without prior review.
- Never skip a security review before a commit.
- Never skip a security review before a deployment.

## Additional resources

- For OWASP and general security reviews: [references/owasp-top-10-2025.md](references/owasp-top-10-2025.md)
- For security reviews of LLM integration: [references/owasp-top-10-2025-LLM.md](references/owasp-top-10-2025-LLM.md)
- Scaffolding script: `${CLAUDE_SKILL_DIR}/scripts/generate.sh`
