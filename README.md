# n8n-workflows

A canonical collection of documented and sanitized [n8n](https://n8n.io)
workflow exports.

Workflows are organized by the project they support. Project repositories link
to the canonical copies here instead of maintaining duplicate exports that can
drift out of date.

## Collections

| Collection | Description |
| --- | --- |
| [`chris-guru`](./chris-guru/) | Automations that publish portfolio activity, maintain the knowledge-vector index, and power the Ziggy chatbot for [chris.guru](https://github.com/christopherjnelson/chris.guru). |
| [`resume-tailor`](./resume-tailor/) | AI-powered resume builder that scrapes a job posting, validates it, retrieves verified candidate data from Notion, and generates a tailored ATS-friendly resume as a PDF. |
| [`sleeper`](./sleeper/) | Fantasy-football reports and digests powered by the read-only [`n8n-nodes-sleeper`](https://github.com/christopherjnelson/n8n-nodes-sleeper) community node. |

Each collection has an inventory README, and each workflow directory contains
one sanitized `workflow.json` export plus its own setup guide.

## Security Policy

Published exports must not contain credential bindings; API keys; OAuth tokens;
passwords, secrets, or authorization headers; private data-source IDs;
production webhook paths; pinned execution data; or n8n instance IDs. Document
required credential types and replace private identifiers with descriptive
placeholders so users reconnect their own services after import.
