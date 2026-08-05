# Discord Workflows

These sanitized workflows provide HTTP endpoints and reusable capabilities for a
Discord bot or integration. The Discord application itself is outside n8n: it
must authenticate to the webhook endpoints, shape the request bodies documented
below, and deliver n8n's responses back to Discord.

## Workflow Inventory

| Workflow | Export | Setup guide | Purpose |
| --- | --- | --- | --- |
| Generate Image Capability | [Export](./capability_generate_image/workflow.json) | [Setup guide](./capability_generate_image/README.md) | Reusable OpenRouter image-generation sub-workflow that returns binary image data. |
| Image Command | [Export](./image_command/workflow.json) | [Setup guide](./image_command/README.md) | Authenticated webhook adapter for a Discord image command. |
| LLM Assistant — Ask | [Export](./llm_assistant_ask/workflow.json) | [Setup guide](./llm_assistant_ask/README.md) | Small authenticated question-answering endpoint with web search. |
| LLM Assistant — Dynamic | [Export](./llm_assistant_dynamic/workflow.json) | [Setup guide](./llm_assistant_dynamic/README.md) | Context-aware Discord assistant with search, scraping, and PostgreSQL memory. |

The private shared Error Workflow from the source instance is intentionally not
published because it contains only instance-specific notification plumbing.

## Security

All inbound webhooks use header authentication in the source instance. The
checked-in exports retain the authentication mode but remove credential bindings
and replace production paths and linked workflow IDs with placeholders. Use a
long, unique webhook path as defense in depth, store caller secrets in n8n
credentials, validate and limit request sizes in the Discord adapter, and never
place bot tokens or webhook secrets in workflow parameters.
