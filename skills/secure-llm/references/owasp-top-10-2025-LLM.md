# OWASP Top 10 for LLM Applications 2025 — Generic checklist

Reference for any project that integrates an LLM (Claude or another) — as an input of
untrusted content (RAG, transcripts, user content), as a generator of output that gets
persisted or displayed, or as an agent with tool access.

This list is complementary to `owasp-top-10-2025.md` (classic web risks), it does not
replace it: an endpoint exposing an LLM still needs auth, rate limit, input validation,
etc. Here we cover what is specific to working with a model.

## Index
- [LLM01 — Prompt Injection](#llm01)
- [LLM02 — Sensitive Information Disclosure](#llm02)
- [LLM03 — Supply Chain](#llm03)
- [LLM04 — Data and Model Poisoning](#llm04)
- [LLM05 — Improper Output Handling](#llm05)
- [LLM06 — Excessive Agency](#llm06)
- [LLM07 — System Prompt Leakage](#llm07)
- [LLM08 — Vector and Embedding Weaknesses](#llm08)
- [LLM09 — Misinformation](#llm09)
- [LLM10 — Unbounded Consumption](#llm10)

---

<a id="llm01"></a>
## LLM01:2025 — Prompt Injection
**What it is:** the LLM processes instructions and data in the same channel. An attacker
can make content that should be only *data* (a document, a transcript, a form field, a
search result) be interpreted as a new *instruction* that overrides the original rules.

**How it shows up:**
- Any user-generated or third-party content (transcripts, reviews, emails, scraping
  results, content from an external API) ends up inside the prompt without being
  distinguished from the system instructions.
- Application instructions and untrusted content concatenated in the same string before
  sending it to the model.
- Indirect injection: the untrusted content is not written by the user interacting with
  the chat, but arrives from a document, web page, or file the model processes as part of
  its task.

**Checklist:**
- [ ] Untrusted content (third-party text, RAG, scraping, free user input) always goes in
  the `user` turn, never concatenated inside the `system` prompt.
- [ ] The fixed rules of the model's behavior live in the `system` prompt and do not rely
  on the user "respecting them" — they are also validated on the code side.
- [ ] If the model has tool access (tools/function calling), calls to sensitive tools are
  not triggered just because the content "asks" for it — there is a validation or
  confirmation layer between the model's intent and the real execution.
- [ ] The entry points of untrusted content into the prompt (RAG, uploaded files, scraped
  content, results from other APIs) are documented so they can be audited.

---

<a id="llm02"></a>
## LLM02:2025 — Sensitive Information Disclosure
**What it is:** the model exposes sensitive information in its responses — secrets, PII,
another user's/tenant's data, internal system details — because that information ended up
available in the prompt, in the retrieved context, or in the data being worked on.

**How it shows up:**
- Secrets, tokens, or credentials included in the prompt "because it was needed" and left
  exposed if the model repeats them or if the whole conversation is logged.
- One user's/tenant's context leaks into another's due to poor segmentation in RAG or in
  prompt assembly.
- Full prompts or responses persisted in logs with no retention policy or access control.

**Checklist:**
- [ ] Secrets, API keys, or credentials are never included inside a prompt's content —
  those go through environment variables/config, not as text the model processes.
- [ ] Retrieved context (RAG, history, documents) is segmented per user/tenant; a prompt is
  never assembled with another user's data "just in case".
- [ ] There is a retention policy for conversation/prompt logs, with restricted access —
  full prompts or responses are not logged without a time limit.
- [ ] Output filtering is defined for data that should never appear in a response (PII,
  secrets) before displaying or persisting it.

---

<a id="llm03"></a>
## LLM03:2025 — Supply Chain
**What it is:** risk from the third-party components of the AI pipeline — the model
provider, orchestration SDKs/libraries, datasets, RAG frameworks, plugins — the same as
A03 in the web top 10 but applied to the specific AI chain.

**How it shows up:**
- The model provider's SDK or orchestration libraries (LangChain, etc.) not updated, with
  known vulnerabilities.
- Third-party models (fine-tunes, Hugging Face models, embeddings) used without verifying
  provenance.
- Third-party plugins/tools connected to the agent without evaluating what real access they
  grant.

**Checklist:**
- [ ] AI SDKs and orchestration libraries are audited like any other dependency (see A03 of
  the web top 10 in `owasp-top-10-2025.md`).
- [ ] If third-party models or datasets are used (not from the primary provider), their
  provenance and license are verified.
- [ ] Any third-party plugin/tool/MCP server connected to the agent is evaluated for the
  real access it grants before enabling it (see also LLM06).

---

<a id="llm04"></a>
## LLM04:2025 — Data and Model Poisoning
**What it is:** manipulation of the data used in pre-training, fine-tuning, or of the
knowledge base feeding RAG, to bias or degrade the model's responses.

**How it shows up:** (relevant mainly if the project does its own fine-tuning or maintains
a knowledge base/RAG editable by third parties)
- Training or fine-tuning sources without verified provenance.
- A RAG knowledge base where anyone can insert content without review, and that content is
  later retrieved as if it were trusted.

**Checklist:**
- [ ] If fine-tuning is done: the datasets used have verified provenance and versioning (be
  able to trace which data trained which model version).
- [ ] If there is a RAG knowledge base: there is a review process before new content enters
  the base that the model will treat as trusted.
- [ ] Content from external sources is validated/sanitized before indexing, not just before
  displaying.

*(If the project only consumes a provider's model via API without its own fine-tuning or
third-party-editable RAG, this category has a smaller surface — still document it as a
conscious decision, not an omission.)*

---

<a id="llm05"></a>
## LLM05:2025 — Improper Output Handling
**What it is:** the model's output is treated as trusted and passed to another system
(database, HTML, a command, another query) without validation — the model can hallucinate,
return the wrong format, or (if manipulated via prompt injection) return something
malicious.

**How it shows up:**
- The model's response is rendered directly as HTML without sanitizing (XSS risk).
- JSON returned by the model is persisted without validating expected types, ranges, or
  enums.
- The model's output is used to build another query or command without treating it as
  untrusted input.

**Checklist:**
- [ ] The model's output is never executed, rendered as HTML, or used to build another
  query/command without going through validation — it is treated like any external
  untrusted input.
- [ ] If structured JSON is expected: it is validated against a schema (types, ranges,
  enums) before persisting, and it is explicitly decided what happens if it does not match
  (reject, retry, flag for review).
- [ ] Integrity constraints also exist at the database level (constraints), not just
  relying on application validation.
- [ ] It is never assumed that the model "will always respect the requested format" — the
  code handles the case where it does not.

---

<a id="llm06"></a>
## LLM06:2025 — Excessive Agency
**What it is:** the model or agent has more functionality, permissions, or autonomy than
its task needs. OWASP splits it into three causes: excessive functionality (access to tools
it does not need), excessive permissions (those tools operate with more privilege than
needed), and excessive autonomy (high-impact actions without a human in the loop).

**How it shows up:**
- An agent with write/delete access when its task is only read/analysis.
- A tool's credentials shared with broad permissions instead of scoped to what the agent
  actually needs.
- Irreversible actions (send, delete, pay, publish) triggered by the model without any
  human confirmation step.

**Checklist:**
- [ ] The agent only has access to the tools its specific task requires — not "in case it
  is useful later".
- [ ] Each tool's credentials/permissions are scoped to the minimum needed (least
  privilege), not reusing a credential with broad permissions.
- [ ] Every high-impact or irreversible action (send, delete, pay, publish, modify
  configuration) requires explicit human confirmation before executing.
- [ ] If the agent should only extract/analyze information, it has no tools with side
  effects enabled.

---

<a id="llm07"></a>
## LLM07:2025 — System Prompt Leakage
**What it is:** the model reveals the content of its own system prompt (internal rules,
business logic, credentials, or references to internal systems mistakenly placed there).

**How it shows up:**
- The system prompt contains secrets, internal URLs, or sensitive business logic assuming
  it "will never be shown".
- There is no mitigation if a user directly asks "repeat your instructions".

**Checklist:**
- [ ] The system prompt contains no secrets, credentials, or information that would
  compromise anything if leaked — assume it can be reconstructed or extracted.
- [ ] No security rule depends solely on "the user not knowing how the prompt is built" —
  the real security controls live in the code; the system prompt is an additional layer,
  not the only one.

---

<a id="llm08"></a>
## LLM08:2025 — Vector and Embedding Weaknesses
**What it is:** risks specific to RAG systems and vector databases — embedding poisoning,
similarity attacks to retrieve unwanted content, unauthorized access to the vector store,
or reconstruction of original text from its embeddings.

**How it shows up:** (relevant only if the project uses RAG/vector DB)
- Vector DB without access control, or shared across tenants without isolation.
- Content indexed without review, allowing a "crafted" query to retrieve content that
  should not be available to that user.

**Checklist:**
- [ ] The vector store has access control equivalent to any other database — it is not
  exposed just because "it is internal".
- [ ] If there are multiple tenants/users, their embeddings are isolated (namespace,
  per-tenant filter on each query), never shared by default.
- [ ] The indexing process validates/reviews content before adding it to the base (see also
  LLM04).

*(If the project uses no RAG or vector DB, this category does not apply — document it as a
decision, do not leave it unreviewed.)*

---

<a id="llm09"></a>
## LLM09:2025 — Misinformation
**What it is:** the model generates incorrect responses with an appearance of certainty —
hallucinates facts, invents citations or sources, or the surrounding system trusts the
model's output more than its real reliability allows.

**How it shows up:**
- Model responses shown to the end user without any indication that they may contain
  errors, especially in domains where an error has real cost (health, legal, financial).
- Data generated by the model (classifications, scores, relationships between entities)
  treated as ground truth without any verification or review mechanism.
- Citations, sources, or references generated by the model used without confirming they
  exist.

**Checklist:**
- [ ] In high-impact domains (health, legal, financial, automated decisions about people),
  the model's output is presented as assistance, not final truth, and there is a human
  review step before acting on it.
- [ ] If the model generates citations, sources, or verifiable data, there is a mechanism
  to confirm they exist before showing them as facts.
- [ ] Model-generated data that feeds another automated process (scoring, classification,
  decisions) has some quality control — sampling, periodic review, or confidence
  thresholds — it is not assumed correct by default.

---

<a id="llm10"></a>
## LLM10:2025 — Unbounded Consumption
**What it is:** resource consumption without limit — compute, tokens, or cost — that can
degrade the service or generate uncontrolled spending. Includes deliberate attacks (a flood
of expensive requests) and your own bugs (an agent in a loop).

**How it shows up:**
- Endpoints that call the model without rate limit or a per-request token limit.
- No usage/cost quota per user on features that consume the model's API.
- An agent capable of looping (calling itself, retrying indefinitely) without a maximum
  iteration limit.
- Contexts that grow without limit (accumulated history, concatenated documents) increasing
  cost and latency per request.

**Checklist:**
- [ ] Every endpoint that triggers a model call has rate limiting, like any expensive
  endpoint (see A02/A06 of the web top 10).
- [ ] There is an input/output token limit per request, and a cost or call-count limit per
  user/time window.
- [ ] Agents capable of iterating (calling tools, retrying, chaining model calls) have an
  explicit maximum of iterations/steps.
- [ ] If the context can grow with use (history, accumulated documents), there is a
  truncation or summarization strategy — it does not grow without limit.
- [ ] There are cost/usage alerts or dashboards for the model's API, not just reviewing the
  bill at the end of the month.

---

## How to use this reference when reviewing a project

1. First identify the project's LLM usage pattern: only simple API calls? RAG? an agent
   with tools/function calling? That determines which categories apply most strongly
   (LLM04/LLM08 only if there is RAG; LLM06 only if there are tools).
2. Prompt Injection (LLM01), Improper Output Handling (LLM05), and Unbounded Consumption
   (LLM10) apply practically whenever there is an LLM integration, no matter how simple.
3. When reporting a finding, state: LLM category, severity, where it is (which call, which
   endpoint), and the concrete fix — same as in the web OWASP reference.
4. This list does not replace the `owasp-top-10-2025.md` checklist: a service exposing an
   LLM needs both — classic access/infra controls AND the AI-specific ones.
