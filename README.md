# secure-develop

A Claude Code plugin for **secure development and security auditing** across backend,
frontend, and infrastructure code.

It bundles a cross-cutting security skill and a dedicated audit agent, both driven by
concrete checklists based on the **OWASP Top 10:2025** and the **OWASP Top 10 for LLM
Applications 2025**. Findings are always anchored to a real file and line — no generic
advice.

## What's included

| Component | Type | Purpose |
| --------- | ---- | ------- |
| `secure-develop` | Skill | Cross-cutting guidance loaded while writing/reviewing code that touches input, data, secrets, or an LLM. Loads only the relevant OWASP reference on demand. |
| `security-auditor` | Agent | On-demand deep audit of code, a PR, or a full project. Produces a severity-ranked report. |
| `owasp-top-10-2025.md` | Reference | Per-category checklist for classic web risks (access control, injection, crypto, logging, etc.). |
| `owasp-top-10-2025-LLM.md` | Reference | Per-category checklist for LLM-specific risks (prompt injection, output handling, excessive agency, unbounded consumption, etc.). |
| `secure-checklist-template.md` | Asset | Report template used when a written security review is requested. |

Target stacks: Java/Quarkus, Spring Boot, Node/NestJS, Next.js, ReactJS, and Docker
containers — with extra focus on apps that integrate LLMs.

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

**The skill** activates automatically while you write or review security-sensitive code
(endpoints/controllers, data access, secret and env-var handling, external calls), or when
you mention `security`, `OWASP`, `injection`, `auth`, `authorization`, `secrets`,
`prompt injection`, `validation`, or `rate limit`. Triggers work in both English and
Spanish.

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

- [OWASP Top 10:2025](skills/secure-develop/references/owasp-top-10-2025.md)
- [OWASP Top 10 for LLM Applications 2025](skills/secure-develop/references/owasp-top-10-2025-LLM.md)

## License

MIT
