---
name: secure-llm
description: Secure development for applications that integrate LLMs, based on OWASP Top 10 for LLM Applications 2025. Use when writing or reviewing code that builds prompts, calls an LLM/agent, handles model output, exposes tools/functions to a model, or implements RAG. Triggers on "prompt injection"/"inyección de prompt", "LLM", "agent"/"agente", "RAG", "tool calling", "excessive agency", "model output", "system prompt", "jailbreak".
allowed-tools: Read, Grep, Glob
---

# Secure LLM Development (OWASP Top 10 for LLM Applications 2025)

Read it before writing or reviewing any code that builds prompts, calls an LLM or agent,
handles model output, exposes tools/functions to a model, or implements RAG.

## When it activates

Use it whenever the code integrates an LLM (Claude or any other): prompt construction,
model/agent calls, handling of model output, tool/function exposure, or RAG pipelines.
Also on mentions of "prompt injection", "LLM", "agent", "RAG", "tool calling",
"excessive agency", "system prompt", or "jailbreak". Triggers work in English and Spanish.

For classic web risks (access control, injection, crypto, auth) use the **secure-web**
skill — LLM apps need both: they are web apps that also happen to call a model.

## Instructions

1. Confirm the app actually calls an LLM (search for provider SDKs: `@anthropic-ai`,
   `openai`, `langchain`, `llamaindex`, etc.). If it does not, this skill does not apply —
   say so and defer to **secure-web**.
2. Load the checklist on demand: [references/owasp-top-10-2025-LLM.md](references/owasp-top-10-2025-LLM.md).
   Only load the categories that apply (e.g. skip LLM08 Vector/Embedding weaknesses if there
   is no RAG/vector DB — but say so explicitly instead of silently omitting it).
3. Apply the checklist against the real code; read context before reporting.
4. Report findings with severity and a concrete fix anchored to `file:line`.

## Hard rules (never negotiate)

- Never trust model output as if it were trusted code or data — treat it as untrusted input
  (it can be the vehicle for injection downstream: SQL, shell, HTML, further prompts).
- Never give a model tool access broader than the task needs (least privilege / agency).
- Never put secrets, keys, or other users' data into a prompt or system prompt.
- Never render raw model output as HTML/Markdown without sanitizing (output handling → XSS).
- Never skip input/output limits — unbounded consumption is a real DoS and cost vector.

## Additional resources

- OWASP Top 10 for LLM Applications 2025: [references/owasp-top-10-2025-LLM.md](references/owasp-top-10-2025-LLM.md)
