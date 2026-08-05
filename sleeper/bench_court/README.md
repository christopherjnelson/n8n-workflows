# Sleeper Bench Court Workflow

This public n8n form accepts a Sleeper league ID and completed NFL week, computes
the highest-scoring legal lineup for every roster, and renders a courtroom-style
report about lineup efficiency, bench points, and results that could have flipped.

## Workflow Overview

```text
[Bench Court Form] → [Log Request] → [Validate]
        → [League] → [Managers] → [Rosters] → [Week Matchups]
        → [Active Player Map] → [Solve Legal Lineups] → [Show Report]
```

The optimizer uses the league's configured starter slots and Sleeper fantasy
position eligibility, including common flex, superflex, and IDP slots. It uses
recorded weekly points and does not claim those outcomes were predictable.

## Requirements and setup

- A self-hosted n8n instance with `n8n-nodes-sleeper@next` installed
- An n8n data table with string `league_id`, number `week`, and date/time
  `submitted_at` columns
- No Sleeper credential; the workflow reads public Sleeper API data

1. Import [workflow.json](./workflow.json).
2. Replace `YOUR_BENCH_COURT_REQUEST_TABLE_ID` by selecting the request-log
   data table.
3. Replace `your-sleeper-bench-court-form-path` with a unique form path.
4. Decide whether the public form needs n8n form authentication.
5. Test a completed week and verify lineup eligibility and the rendered report
   before publishing.

The log stores submitted league IDs and timestamps. Set retention and access
controls accordingly. The export contains no real form path, table ID, pinned
requests, credential binding, or instance ID.
