# Daily Sleeper Trends Workflow

This n8n workflow publishes a daily Discord digest of the ten most-added and
ten most-dropped NFL players over the previous 24 hours. It runs manually or at
9:00 AM in the workflow timezone and imports inactive.

## Workflow Overview

```text
[Manual / Daily 9 AM Trigger]
             ↓
    [Get Trending Adds]
             ↓
    [Get Trending Drops]
             ↓
 [Get Active Player Map]
             ↓
   [Build Discord Digest]
             ↓
    [Send to Discord]
```
<img width="1691" height="651" alt="n8n workflow canvas for the Sleeper daily trends" src="https://github.com/user-attachments/assets/7119b573-a959-420b-ab84-bd9234fd7737" />

## How It Works

1. Requests ten trending adds and ten trending drops from the prior 24 hours.
2. Fetches the active NFL player map once and resolves each returned player ID
   to a name, position, team, and available injury status.
3. Escapes Discord mention and Markdown characters in Sleeper-supplied values.
4. Builds separate add and drop embeds, suppresses push notifications, and
   includes the required Sleeper attribution.
5. Sends the digest through a Discord incoming webhook.

<img width="639" height="737" alt="Discord message containing Sleeper trending adds and drops" src="https://github.com/user-attachments/assets/b6b85152-a741-4c03-ada9-6b97586c1358" />


The player operation returns raw Sleeper IDs and counts, so the separate player
map lookup is intentional. Keep this workflow at a daily cadence unless you
replace that lookup with an external cache.

## Requirements

- A self-hosted n8n environment with
  [`n8n-nodes-sleeper`](https://github.com/christopherjnelson/n8n-nodes-sleeper)
  installed
- A Discord incoming webhook and n8n **Discord Webhook API** credential

Sleeper API access itself needs no credential. The custom node is unofficial,
read-only, and currently released as the `0.1.0` community-testing prerelease.

## Import and Configuration

1. Install the custom node using its
   [installation guide](https://github.com/christopherjnelson/n8n-nodes-sleeper#installation).
2. Import [workflow.json](./workflow.json) into n8n.
3. Create or select a Discord Webhook API credential on **Send Digest to
   Discord**. The checked-in export intentionally has no credential binding.
4. Confirm **Daily at 9 AM** uses the intended workflow timezone.
5. Run **Test Workflow Manually** and verify the names, counts, embeds, and
   Sleeper attribution in the target channel.
6. Publish or activate the workflow when ready.

The Discord webhook is a secret. Keep its URL only in the n8n credential store
and never add it directly to the workflow JSON.
