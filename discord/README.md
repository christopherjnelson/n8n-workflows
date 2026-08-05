# Discord Workflows

These sanitized workflows provide HTTP endpoints and reusable capabilities for a
Discord bot or integration. The Discord application itself is outside n8n: it
must authenticate to the webhook endpoints, shape the request bodies documented
below, and deliver n8n's responses back to Discord.

## Workflow Inventory

| Workflow | Export | Setup guide | Purpose |
| --- | --- | --- | --- |
| Discord — Ask Assistant | [Export](./ask-assistant/workflow.json) | [Setup guide](./ask-assistant/README.md) | Small authenticated question-answering endpoint with web search. |
| Discord — Contextual Assistant | [Export](./contextual-assistant/workflow.json) | [Setup guide](./contextual-assistant/README.md) | Context-aware Discord assistant with search, scraping, and PostgreSQL memory. |
| Discord — Generate Image Capability | [Export](./generate-image-capability/workflow.json) | [Setup guide](./generate-image-capability/README.md) | Reusable OpenRouter image-generation sub-workflow that returns binary image data. |
| Discord — Image Command | [Export](./image-command/workflow.json) | [Setup guide](./image-command/README.md) | Authenticated webhook adapter for a Discord image command. |

The private `Discord — Workflow Error Alerts` workflow from the source instance
is intentionally not published because it contains only instance-specific
notification plumbing.

## Security

All inbound webhooks use header authentication in the source instance. The
checked-in exports retain the authentication mode but remove credential bindings
and replace production paths and linked workflow IDs with placeholders. Use a
long, unique webhook path as defense in depth, store caller secrets in n8n
credentials, validate and limit request sizes in the Discord adapter, and never
place bot tokens or webhook secrets in workflow parameters.
