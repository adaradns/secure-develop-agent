# secure-develop

A Claude Code plugin for **secure development and security auditing** across backend,
frontend, and infrastructure code.

It bundles three topic-scoped security skills and a dedicated audit agent, all driven by
concrete checklists based on the **OWASP Top 10:2025** and the **OWASP Top 10 for LLM
Applications 2025**. Findings are always anchored to a real file and line — no generic
advice.

## What's included

Skills are split by **topic** (the activation axis); the **stack** (language/framework) is
resolved inside each skill by loading the matching reference on demand — not by adding more
skills.

| Component | Type | Purpose |
| --------- | ---- | ------- |
| `secure-web` | Skill | OWASP Top 10:2025 for application code — endpoints, data access, auth, secrets. Loads a stack guide (NestJS / Next.js / Spring Boot / Quarkus) on demand. |
| `secure-llm` | Skill | OWASP Top 10 for LLM Applications 2025 — prompt injection, output handling, excessive agency, unbounded consumption, RAG. |
| `secure-infra` | Skill | Dockerfiles & container images, Kubernetes/Helm manifests, and CI/CD pipelines. |
| `security-auditor` | Agent | On-demand deep audit of code, a PR, or a full project. Produces a severity-ranked report, picking the references above that apply. |
| `secure-checklist-template.md` | Asset | Report template used when a written security review is requested (in `secure-web/assets/`). |

Target stacks: Java/Quarkus, Spring Boot, Node/NestJS, Next.js, and Docker/Kubernetes/CI-CD
— with extra focus on apps that integrate LLMs.

## Installation

This repository is a self-contained Claude Code plugin marketplace. Add it, then install
the plugin:

```
/plugin marketplace add adaradns/secure-develop-agent
/plugin install secure-develop@secure-develop-agent
```

Or point Claude Code at a local checkout during development:

```
/plugin marketplace add /path/to/secure-develop-agent
/plugin install secure-develop@secure-develop-agent
```

Restart or reload Claude Code so the skill and agent are registered. Manage or remove it
later with `/plugin`.

## How it activates

**The skills** activate automatically by topic while you write or review code:
- `secure-web` on endpoints/controllers, data access, secret/env handling, external calls,
  or mentions of `security`, `OWASP`, `injection`, `auth`, `authorization`, `secrets`,
  `validation`, `rate limit`, CORS, CSRF.
- `secure-llm` on code that builds prompts or calls an LLM/agent, or mentions of
  `prompt injection`, `LLM`, `agent`, `RAG`, `tool calling`, `excessive agency`.
- `secure-infra` on Dockerfiles, compose files, Kubernetes/Helm manifests, or CI/CD
  workflows, or mentions of `docker`, `container`, `kubernetes`, `CI/CD`, `pipeline`.

Triggers work in both English and Spanish.

**The agent** is for a thorough audit rather than a one-off question. Invoke it when you
want a full review:

```
Use the security-auditor agent to review this project before we deploy.
Audit the security of the /api folder.
```

For a single "is this line safe?" question, the skill answers directly without spawning
the agent.

## Example

```
> Review the security of src/routes/orders.ts before I merge.
```

The auditor scopes the files, picks the applicable references, greps for risk patterns,
verifies each hit against the real code, and returns findings like:

```
## [CRITICAL] Missing ownership check on order lookup
**Category:** OWASP A01
**Where:** src/routes/orders.ts:42
**What happens:** GET /orders/:id returns any order by id without checking it belongs to
the authenticated user.
**Fix:** filter the query by the authenticated user id, or return 404 when ownership fails.
```

...followed by an executive summary of counts per severity and the top items to fix first.

## Design principles

- **Evidence over advice.** Every finding is anchored to a real file:line, verified in
  context — a grep match alone is not a finding.
- **No severity inflation.** A low issue mislabeled as critical wastes the reader's time.
- **Load on demand.** References are read only when relevant, not all at once.
- **Never leak secrets.** Findings report that a secret exists and where — never its value.

## References

- [OWASP Top 10:2025](skills/secure-web/references/owasp-top-10-2025.md) — generic checklist
  - Stack guides: [NestJS](skills/secure-web/references/stacks/nestjs.md) · [Next.js](skills/secure-web/references/stacks/nextjs.md) · [Spring Boot](skills/secure-web/references/stacks/spring-boot.md) · [Quarkus](skills/secure-web/references/stacks/quarkus.md)
- [OWASP Top 10 for LLM Applications 2025](skills/secure-llm/references/owasp-top-10-2025-LLM.md)
- Infrastructure: [Dockerfile](skills/secure-infra/references/dockerfile.md) · [Kubernetes](skills/secure-infra/references/kubernetes.md) · [CI/CD](skills/secure-infra/references/ci-cd.md)

## License

MIT
