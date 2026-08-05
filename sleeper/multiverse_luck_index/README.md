# Sleeper Multiverse — Discord Weekly Luck Index

This scheduled workflow replays every team against every other score from a
Sleeper league week. It compares each team's all-play win rate with the real
matchup result, ranks the luck swing, and sends a three-embed Discord report.

## Workflow Overview

```text
[Manual / Tuesday 9 AM] → [Configure] → [NFL State]
  → [Resolve Week] → [League] → [Managers] → [Rosters]
  → [Selected + Previous Week Matchups] → [Build Luck Index]
  → [Send Discord Embeds]
```

With no manual week override, the workflow checks the current Sleeper week and
falls back to the previous week when scoring has not started. It returns no
items—and sends nothing—when neither dataset contains usable scores.

## Requirements and setup

- A self-hosted n8n instance with `n8n-nodes-sleeper@next` installed
- A Discord incoming webhook configured as an n8n Discord Webhook credential
- No Sleeper credential

1. Import [workflow.json](./workflow.json).
2. In **Configure Multiverse**, replace `YOUR_SLEEPER_LEAGUE_ID` with the
   season-specific league ID.
3. Leave `weekOverride` at `0` for scheduled operation or set 1–18 for a
   historical manual test.
4. Attach the Discord Webhook credential to **Post Multiverse to Discord**.
5. Confirm the workflow timezone and Tuesday 9:00 AM schedule.
6. Run the manual trigger, review all embeds and mention suppression, then
   publish only when ready.

The export contains no real league ID, Discord credential binding, pinned data,
or n8n instance ID.
