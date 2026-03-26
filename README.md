# Prismm · Sequence Review Portal

**URL:** https://keyona-rerev.github.io/prismm-sequence-review
**Password:** prismm2026
**Repo:** `keyona-rerev/prismm-sequence-review`

---

## TL;DR — Quick Usage Reference

| I want to… | Do this |
|---|---|
| Push a new sequence for review | Run the sequence push workflow (GAS system) — it updates `sequences.json` automatically |
| See what's awaiting review | Open the dashboard → Sequences tab → filter = Awaiting Review |
| Add an institution to the pipeline | Edit `pipeline.json` → add object to `institutions` array with correct `bucket` value |
| Move an institution between columns | Change its `bucket` value in `pipeline.json` |
| Add a weekly report | Create `reports/week-N.json` (e.g. `reports/week-1.json`) in the repo |
| Update batch name / strategic header | Edit `meta.current_batch` in `pipeline.json` |
| Add a new scenario or audience profile | Edit `scenarios.json` or `audiences.json` directly |
| Change system info (personas, domains, costs) | Edit the hardcoded content in `index.html` → `renderSysInfo()` function |

---

## What This Is

A password-gated GitHub Pages dashboard that serves as the operational hub for Prismm's cold email sales system. It has two audiences:

- **Keyona** — monitors pipeline, reviews sequences, tracks weekly performance
- **Martha / Shella** — review email sequences before they go live, leave feedback via GitHub Issues

Everything is static — no backend, no database. All data lives in JSON files in this repo. The dashboard fetches those files on load and renders them client-side.

---

## Tab Overview

### Sequences
The core review workflow. Reads from `sequences.json` and `rejections.json`.

A filter dropdown at the top lets you view: All / Awaiting Review / Approved / Rejected. Default view is **Awaiting Review**. Each card shows a status pill (amber = pending, green = approved, red = rejected).

**You do not manually edit this tab's data.** The GAS sequence push system writes to `sequences.json` and `rejections.json` automatically. Martha and Shella review here and leave feedback via the GitHub Issues link on each card.

The tab badge shows the pending count only — it's the number that matters most.

---

### Pipeline
Tracks which institutions are in your active outreach. Reads from `pipeline.json`.

Three columns:
- **Warming Up** — institutions you're researching or warming up to before contact
- **Batch A Active** — currently in an active sequence
- **Batch B Queued** — ready to go, waiting for Batch A to clear

The header shows the current batch name and a strategic statement derived from `meta`. Cards show: institution name, location, asset size, tier, any connection flags, and signals.

**You manually maintain `pipeline.json`.** See the Pipeline section below for the full data format.

---

### Weekly Reports
Performance recap by week. Reads from `reports/week-N.json` files (e.g. `reports/week-1.json`, `reports/week-2.json`).

The dashboard probes for these files sequentially on load — starting at week-1, stopping at the first missing file. The dropdown defaults to the latest week found. If no report files exist, the tab shows a clean empty state.

**You create one JSON file per week.** See the Weekly Reports section below for the schema.

> ⚠️ Do not skip week numbers. If you have week-1 and week-3 but not week-2, the dashboard stops loading at week-2 and week-3 never appears.

---

### Scenarios
Documents the defined outreach scenarios (e.g. "Cold CB VP/EVP Southeast"). Reads from `scenarios.json`. Each scenario links to one or more sequences by ID.

You edit this file when you define a new strategic scenario — a new audience segment, trigger condition, or use case that warrants its own sequence track.

---

### Audiences
Documents your ICP profiles — who you're targeting, their titles, asset size, geography, pain point, and outreach angle. Reads from `audiences.json`. Each audience links to sequences by ID.

You edit this file when you add a new ICP segment or refine an existing one.

---

### System Info
Hardcoded reference panel. Shows: personas (Roland, Keyona, Martha), sending domains and their status, monthly cost estimate, warm-up schedule, lead flow diagram.

This does not read from a JSON file. To change it, edit the `renderSysInfo()` function directly in `index.html`.

---

## File Map

```
prismm-sequence-review/
│
├── index.html                  # The entire dashboard — all tabs, styles, JS
│
├── sequences.json              # All sequences + their status (pending/approved/rejected)
├── rejections.json             # Rejection reasons keyed by sequence ID
├── scenarios.json              # Outreach scenario definitions
├── audiences.json              # ICP audience profiles
├── pipeline.json               # Pipeline institutions by bucket/column
│
├── reports/
│   ├── week-1.json             # Weekly performance report (create manually)
│   ├── week-2.json             # Add one per week, no gaps, no zero-padding
│   └── ...
│
├── sequences/
│   ├── seq-001-*.html          # Full rendered sequence files (pushed by GAS system)
│   └── seq-002-*.html
│
├── templates/
│   └── plain-sequence-template.html   # Template used by GAS push workflow
│
└── data/
    ├── credit-unions.csv       # Full NCUA credit union dataset (used historically)
    └── targets.json            # Legacy targets intel (no longer drives a tab)
```

---

## Data Formats

### `sequences.json`
```json
{
  "sequences": [
    {
      "id": "seq-001",
      "title": "Cold Outreach — VP/EVP Southeast",
      "targetAudience": "VP/EVP Digital Banking, Southeast community banks",
      "status": "pending",
      "pushedDate": "Mar 26, 2026",
      "emailCount": 5,
      "cadence": "Day 1 / 3 / 7 / 14 / 21",
      "sender": "Roland",
      "type": "html",
      "sequenceFile": "sequences/seq-001-cold-cb-vpevp-southeast.html",
      "usageNote": "Optional context note shown on the card.",
      "approvalNote": "Shown on card if status is approved.",
      "rejectionReason": "Shown on card if status is rejected."
    }
  ]
}
```

`status` must be `pending`, `approved`, or `rejected`.

---

### `rejections.json`
```json
{
  "rejections": [
    {
      "id": "seq-001",
      "reason": "Email 3 tone is too pushy — needs softening before send."
    }
  ]
}
```

Rejection reasons here are merged onto sequence cards at render time. Either put the reason here or directly on the sequence object in `sequences.json` — the dashboard checks both.

---

### `pipeline.json`
```json
{
  "meta": {
    "current_batch": "Batch A — Southeast Tier 1",
    "last_updated": "March 26, 2026",
    "total_contacted": 0,
    "tier": 1
  },
  "batches": [],
  "institutions": [
    {
      "name": "Coastal Community Credit Union",
      "city": "Birmingham",
      "state": "AL",
      "assets": 450000000,
      "tier": 1,
      "signals": ["Hiring digital banking role", "New board member from fintech"],
      "connected": true,
      "bucket": "warming_up"
    }
  ]
}
```

**`bucket` values and what they mean:**

| Value | Dashboard column |
|---|---|
| `warming_up` | Warming Up |
| `batch_a_active` | Batch A Active |
| `batch_b_queued` | Batch B Queued |

`assets` is a raw number (not a string). The dashboard formats it automatically — `450000000` renders as `$450M`, `1200000000` renders as `$1.20B`.

`connected: true` adds a purple "✓ Connection" tag to the card.

`signals` is a list of short strings shown at the bottom of each card. Keep them tight — one phrase each.

---

### `reports/week-N.json`
```json
{
  "date_range": "Mar 24 – Mar 30, 2026",
  "stats": {
    "Sent": 42,
    "Open rate": "38%",
    "Replies": 5,
    "Meetings booked": 1
  },
  "sequences": [
    {
      "name": "Cold CB VP/EVP Southeast",
      "sent": 42,
      "opens": 16,
      "replies": 5,
      "notes": "Email 2 outperforming on opens"
    }
  ],
  "highlights": [
    "First reply from Coastal Community CU",
    "38% open rate — above benchmark"
  ],
  "next_steps": [
    "Move Coastal CU to Batch A Active in pipeline.json",
    "Draft email 4 for VP/EVP sequence"
  ],
  "notes": "Free text field — anything that doesn't fit the above."
}
```

All fields are optional except the implicit week number (from the filename). The dashboard renders whatever fields are present and skips the rest cleanly.

---

### `scenarios.json`
```json
{
  "scenarios": [
    {
      "id": "sc-001",
      "name": "Cold Outreach — Community Bank VP/EVP",
      "targetAudience": "VP/EVP Digital Banking",
      "description": "First-touch cold sequence targeting digital banking decision makers at community banks in the Southeast.",
      "trigger": "No prior contact",
      "assetSize": "$250M–$2B",
      "tags": ["Cold", "Southeast", "Community Bank"],
      "typicalTitles": ["VP Digital Banking", "EVP Operations", "Chief Digital Officer"],
      "sequences": ["seq-001", "seq-002"]
    }
  ]
}
```

`sequences` is a list of sequence IDs from `sequences.json`. The Scenarios tab uses these to link through to the actual sequence files.

---

### `audiences.json`
```json
{
  "audiences": [
    {
      "id": "aud-001",
      "name": "Southeast Community Bank VP/EVP",
      "segment": "Community Banking",
      "targetTitles": ["VP Digital Banking", "EVP Operations"],
      "assetSize": "$250M–$2B",
      "geo": "Southeast US",
      "painPoint": "Legacy core systems creating friction in member onboarding.",
      "angle": "Position Prismm as the layer that solves document access without a core replacement.",
      "sequences": ["seq-001"]
    }
  ]
}
```

---

## How the System Fits Together

```
GAS Sequence Push Workflow
        │
        ▼
sequences.json + sequences/*.html   ←── sequences pushed here automatically
        │
        ▼
Dashboard (index.html)
        │
        ├── Sequences tab    reads sequences.json + rejections.json
        ├── Pipeline tab     reads pipeline.json              ←── you maintain manually
        ├── Weekly Reports   reads reports/week-N.json        ←── you create weekly
        ├── Scenarios tab    reads scenarios.json             ←── you maintain manually
        ├── Audiences tab    reads audiences.json             ←── you maintain manually
        └── System Info      hardcoded in index.html
```

The dashboard is fully static — no server, no API. Every time it loads it fetches the latest versions of all JSON files directly from the raw GitHub CDN with a cache-busting timestamp. Changes to any JSON file are live within ~60 seconds of committing.

---

## Workflow: Sequences (How Martha and Shella Use This)

1. Keyona (or the GAS system) pushes a new sequence — `sequences.json` gets a new entry with `status: "pending"` and the rendered HTML file lands in `sequences/`
2. Martha and Shella receive a notification email
3. They open the dashboard, go to Sequences → Awaiting Review
4. They click **View sequence** to read the full email sequence
5. If they have feedback, they click **Leave feedback** — this opens a pre-filled GitHub Issue
6. Keyona reviews feedback, revises if needed, updates `status` to `approved` or `rejected` (and adds `rejectionReason` if rejected) in `sequences.json`

---

## Naming Conventions

| Thing | Convention | Example |
|---|---|---|
| Sequence files | `seq-NNN-slug.html` | `seq-003-warm-handoff-cro.html` |
| Sequence IDs | `seq-NNN` | `seq-003` |
| Scenario IDs | `sc-NNN` | `sc-002` |
| Audience IDs | `aud-NNN` | `aud-003` |
| Weekly report files | `reports/week-N.json` | `reports/week-4.json` |
| Pipeline buckets | snake_case | `warming_up`, `batch_a_active`, `batch_b_queued` |

No zero-padding on week numbers. `week-4.json` not `week-04.json`.
