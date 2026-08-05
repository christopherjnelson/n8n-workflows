# LLM Assistant — Dynamic

A context-aware Discord assistant endpoint. It converts the triggering message
and recent channel context into an agent input, uses an OpenRouter model,
SearXNG search, Firecrawl scraping, and PostgreSQL chat memory, then returns a
Discord adapter response shaped as `{"action":"reply","content":"..."}`.

## Expected request

The request body contains a `trigger` object with author, guild, channel,
message, and content fields plus an optional `context` array of recent
messages. The workflow derives its memory key from the Discord guild and channel
IDs. Recent context is explicitly treated as untrusted text in the agent prompt.

## Setup

1. Import [workflow.json](./workflow.json).
2. Replace `your-llm-assistant-dynamic-webhook-path` with a unique path.
3. Attach credentials for webhook header authentication, OpenRouter, SearXNG,
   Firecrawl, and PostgreSQL chat memory.
4. Confirm the `discord_chat_memory` table name and retention policy are
   appropriate for your database.
5. Review the system prompt, timezone, maximum agent iterations, allowed tools,
   and Discord-side payload limits.
6. Test search, scraping, memory isolation, empty context, and malformed payloads
   before publishing.

Discord content and identifiers can be personal data. Minimize the context sent
to n8n, restrict access to the endpoint, and define retention/deletion behavior
for PostgreSQL memory. The export contains no credential bindings or production
webhook path.
