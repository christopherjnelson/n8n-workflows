# Sleeper Preseason Command Center Workflow

This n8n form workflow accepts a public Sleeper username and league ID, queries
the league's public data, and renders a browser-based preseason command center.
No Sleeper login or Sleeper credential is required.

## Workflow Overview

```text
[Public n8n Form]
       ↓
[Resolve User] → [League] → [League Users] → [Rosters]
       ↓
[League Drafts] → [Traded Picks] → [NFL State]
       ↓
[Build HTML and Markdown Report]
       ↓
[Show Browser Result]
```

## Report Contents

- League identity, season, status, team count, and related IDs
- Current NFL season and week state
- Selected user and matching owned or co-owned roster
- Team owners, roster sizes, reserve and taxi counts, records, and points
- Associated drafts and league-scoped traded draft picks
- Roster positions, scoring highlights, and all league settings

The workflow joins league-user and roster responses explicitly because
[`n8n-nodes-sleeper`](https://github.com/christopherjnelson/n8n-nodes-sleeper)
returns raw API resources without hidden joins. League and user IDs remain
strings throughout the workflow.

## Requirements and Privacy

- A self-hosted n8n environment with `n8n-nodes-sleeper` installed
- Public Sleeper usernames and league IDs supplied by form users
- No n8n or Sleeper credential

The custom node is unofficial, read-only, and currently released as the
`0.1.0` community-testing prerelease. Sleeper data is public, but the generated
report includes usernames, display names, user IDs, league settings, and roster
details. The form is intentionally unauthenticated, so anyone with its URL can
submit requests and view the generated response. Add n8n form authentication
before publishing if that exposure is not appropriate.

## Import and Configuration

1. Install the custom node using its
   [installation guide](https://github.com/christopherjnelson/n8n-nodes-sleeper#installation).
2. Import [workflow.json](./workflow.json) into n8n.
3. Open **Sleeper League Form** and replace
   `your-sleeper-command-center-form-path` with a unique path.
4. Decide whether to enable form authentication and configure it before
   publishing.
5. Test with a public Sleeper username and a league ID from that league's URL.
6. Verify the selected roster, draft ownership, league settings, and browser
   output before publishing the production form.

Sleeper usernames can change. The workflow resolves the submitted username on
each run and uses the stable returned user ID for roster matching.
