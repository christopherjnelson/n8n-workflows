# chris.guru Workflows

These workflows support [chris.guru](https://github.com/christopherjnelson/chris.guru)
by publishing portfolio activity, maintaining the vector knowledge base, and
powering the Ziggy chatbot. This directory is the canonical home for their
sanitized n8n exports and setup instructions.

## Workflow Inventory

| Workflow | Export | Setup guide | Purpose |
| --- | --- | --- | --- |
| Portfolio — Credentials Feed Sync | [Export](./feed-updates/credentials-feed-sync/workflow.json) | [Setup guide](./feed-updates/credentials-feed-sync/README.md) | Polls Credly and Microsoft Learn, publishes new achievements to Supabase, and optionally posts them to LinkedIn. |
| Portfolio — GitHub Feed Sync | [Export](./feed-updates/github-feed-sync/workflow.json) | [Setup guide](./feed-updates/github-feed-sync/README.md) | Summarizes GitHub push events and publishes portfolio feed entries to Supabase. |
| Portfolio — Knowledge Vector Sync | [Export](./knowledge-vector-sync/workflow.json) | [Setup guide](./knowledge-vector-sync/README.md) | Synchronizes public portfolio content from Notion into the Supabase vector knowledge base. |
| Portfolio — Chatbot | [Export](./portfolio-chatbot/workflow.json) | [Setup guide](./portfolio-chatbot/README.md) | Serves Ziggy's chat and health webhooks and answers questions using the vector knowledge base. |

## Directory Structure

```text
chris-guru/
├── README.md
├── feed-updates/
│   ├── credentials-feed-sync/
│   │   ├── README.md
│   │   └── workflow.json
│   └── github-feed-sync/
│       ├── README.md
│       └── workflow.json
├── knowledge-vector-sync/
│   ├── README.md
│   └── workflow.json
└── portfolio-chatbot/
    ├── README.md
    └── workflow.json
```

Each workflow directory contains a sanitized `workflow.json` export and a
`README.md` describing its behavior, dependencies, credentials, and import
steps.

## Publishing and Sanitization Rules

Workflow logic and public portfolio information may be version-controlled. This
includes public profile and repository names, prompts, model identifiers,
database table names, and internal node IDs.

Never publish:

- Credential bindings, credential IDs, or credential names from an n8n instance
- API keys, OAuth tokens, passwords, authorization headers, private keys, or secrets
- n8n instance IDs or production webhook paths and IDs
- Private account, workspace, project, or Notion data-source identifiers
- Pinned execution data or exported secrets files

Credential types and required services may be documented, but users must
reconnect their own credentials after importing an export.

## Safe Export Process

1. Export the workflow JSON from n8n into its workflow directory.
2. Remove every nonempty node `credentials` object.
3. Remove or blank `meta.instanceId`.
4. Replace production webhook paths, webhook IDs, and private data-source IDs
   with descriptive placeholders.
5. Replace linked workflow and data-table IDs, and remove cached resource URLs
   that contain private identifiers.
6. Remove pinned execution data.
7. Validate the JSON and scan it before committing:

   ```bash
   jq -e . workflow.json >/dev/null
   rg -ni "api.?key|authorization|bearer|password|private.?key|secret|token" workflow.json
   ```

8. Review every match manually because security-related words may legitimately
   appear in workflow logic, prompts, or documentation.
9. Update the matching setup guide and review the complete diff.
