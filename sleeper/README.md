# Sleeper Workflows

These workflows turn public Sleeper fantasy-football data into reports and
notifications. They use Christopher Nelson's unofficial, read-only
[`n8n-nodes-sleeper`](https://github.com/christopherjnelson/n8n-nodes-sleeper)
community node and do not require a Sleeper login or Sleeper credential.

## Workflow Inventory

| Workflow | Export | Setup guide | Purpose |
| --- | --- | --- | --- |
| Daily Trends | [Export](./daily_trends/workflow.json) | [Setup guide](./daily_trends/README.md) | Posts the top ten player adds and drops from the previous 24 hours to Discord each morning. |
| Preseason Command Center | [Export](./preseason_command_center/workflow.json) | [Setup guide](./preseason_command_center/README.md) | Accepts a public Sleeper username and league ID, then renders a browser-based league, roster, draft, and scoring report. |

## Community Node Requirements

The workflows use package `n8n-nodes-sleeper` version `0.1.0`, currently a
community-testing prerelease. Install it through **Settings → Community Nodes**
on a supported self-hosted n8n instance, or follow the package repository's
[installation instructions](https://github.com/christopherjnelson/n8n-nodes-sleeper#installation).
The package README documents Node.js 22.22.0 or newer and n8n 2.32.7 as its
tested environment. n8n Cloud availability is not implied.

The node exposes public, deterministic, read-only Sleeper API operations. It
does not perform hidden joins or enrichment, and empty API arrays remain empty.
Usernames may change, while returned user, league, roster, player, and draft IDs
must be treated as opaque strings.

Install package: n8n-nodes-sleeper
npm: https://www.npmjs.com/package/n8n-nodes-sleeper


## Data and Attribution

Sleeper exposes the data used here publicly, but importing and running a
workflow can still store or forward it. Review downstream retention and access
before activation. Any displayed or republished trending-player data must
credit Sleeper; the Daily Trends workflow includes that attribution.

The complete NFL player map is roughly 5 MB. Sleeper recommends fetching it
sparingly and generally no more than once daily. Daily Trends fetches one active
player map per run and uses it only to resolve the trending player IDs.

## Directory Structure

```text
sleeper/
├── README.md
├── daily_trends/
│   ├── README.md
│   └── workflow.json
└── preseason_command_center/
    ├── README.md
    └── workflow.json
```

The checked-in exports contain no Sleeper credentials, n8n instance IDs,
production webhook IDs, pinned execution data, or real form paths. The Discord
workflow's credential binding is also removed and must be reconnected after
import.
