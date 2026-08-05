# LLM Assistant — Ask

A compact authenticated Discord-oriented question-answering endpoint. It accepts
a prompt, runs an AI agent backed by an OpenRouter chat model and SearXNG search,
and returns `{"answer":"..."}`.

## Request contract

POST JSON shaped as `{"prompt":"your question"}` to the configured webhook.
The endpoint is intended for a trusted Discord adapter, not direct anonymous
public use.

## Setup

1. Import [workflow.json](./workflow.json).
2. Replace `your-llm-assistant-ask-webhook-path` with a unique webhook path.
3. Attach an HTTP Header Auth credential to **Webhook**.
4. Attach an OpenRouter credential to **OpenRouter Chat Model**.
5. Configure the SearXNG credential/base URL on **SearXNG**.
6. Review the model, tool access, rate limits, response size, and prompt handling,
   then test through the Discord-side caller before publishing.

Credential bindings and production webhook identifiers are removed from the
public export.
