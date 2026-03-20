# Prismm Sequence Review — Workflow Reference

**Live site:** https://keyona-rerev.github.io/prismm-sequence-review
**Password:** prismm2026
**Recipients:** martha@getprismm.com, shella@getprismm.com

---

## How this works

This repo is the single source of truth for all Prismm email sequences. Every sequence goes through the same pipeline:

```
Draft in conversation → Keyona approves → Claude pushes to repo →
GitHub Pages publishes → Claude emails Martha + Shella →
Martha/Shella review on site → Feedback via GitHub Issues →
Keyona approves or rejects → Claude updates status
```

---

## Starting a new sequence (paste at the top of any drafting conversation)

```
We're drafting a new Prismm email sequence for the review pipeline.

Before writing anything:
1. Check sequences.json and rejections.json in keyona-rerev/prismm-sequence-review
   for patterns in what Martha has rejected — do not repeat those mistakes.
2. Check scenarios.json for the relevant scenario and match the target audience,
   tone, and approach to what's already established.

When I approve the sequence in conversation:
1. Name the file: sequences/seq-[NNN]-[short-slug].html
2. Use templates/plain-sequence-template.html for plain text sequences.
   For HTML email sequences, build from scratch.
3. Add a review bar at the top (status: Awaiting Review, back link to portal).
4. Push the sequence HTML to the repo.
5. Add entry to sequences.json (status: pending, include targetAudience,
   sequenceFile path, scenario tag).
6. Add or update the relevant entry in scenarios.json.
7. Send notification email to martha@getprismm.com and shella@getprismm.com
   via Gmail MCP using the template in this README.
8. Confirm all steps done and give me the direct link to the sequence on Pages.

Sequence formatting rules:
- Plain text: use DM Sans / DM Mono styling from the template. A/B versions per email.
- HTML sequences: full styled email preview.
- Every sequence must include: timeline, per-email rationale, use guidance
  (A/B decision rules), CRM tags table, restart triggers.
- Target audience must be explicit — it surfaces on every card in the review portal.
```

---

## Approving or rejecting a sequence

Tell Claude: **"Approve seq-[NNN]"** or **"Reject seq-[NNN], reason: [reason]"**

Claude will:
- Update `status` in `sequences.json` to `approved` or `rejected`
- Add `approvalNote` or `rejectionReason` field
- Update the review bar in the sequence HTML to reflect new status
- Add rejection reason to `rejections.json` so future sequences learn from it

---

## File structure

```
prismm-sequence-review/
├── index.html                           ← review portal (password gated)
├── sequences.json                       ← all sequence metadata + status
├── scenarios.json                       ← scenario catalog with audience + context
├── rejections.json                      ← rejection history (Claude reads before drafting)
├── robots.txt                           ← blocks all search crawlers
├── sequences/
│   └── seq-001-conference-outreach.html
└── templates/
    └── plain-sequence-template.html     ← base template for plain text sequences
```

---

## sequences.json field reference

```json
{
  "id": "seq-001",
  "title": "Human-readable sequence name",
  "status": "pending | approved | rejected",
  "type": "plain | html",
  "pushedDate": "Mar 20, 2026",
  "approvedDate": "Mar 20, 2026",
  "emailCount": 3,
  "cadence": "6-week cadence",
  "sender": "Martha | Keyona | Dean",
  "targetAudience": "Who this is for — surfaces on every card",
  "sequenceFile": "sequences/seq-001-conference-outreach.html",
  "approvalNote": "Why it worked",
  "rejectionReason": "Why it was rejected",
  "usageNote": "When to use vs not use this sequence"
}
```

---

## scenarios.json field reference

```json
{
  "id": "scenario-001",
  "name": "Scenario name",
  "description": "What this scenario is and when it applies",
  "targetAudience": "Who you're reaching — shown as subtitle on scenario card",
  "typicalTitles": ["CRO", "VP Digital Banking"],
  "assetSize": "$250M–$2B",
  "trigger": "What initiates outreach",
  "sequences": ["seq-001", "seq-002"],
  "tags": ["conference", "credit-union"]
}
```

---

## Notification email template

**Subject:** New sequence ready for review — [Sequence Name]

**Body:**
```
Hi Martha and Shella,

A new email sequence is ready for your review:

[SEQUENCE NAME]
Target audience: [TARGET AUDIENCE]

[CONTEXT: 2–3 sentences — what this sequence is for, why the approach
was chosen, anything Martha needs to know before reviewing]

Review it here: https://keyona-rerev.github.io/prismm-sequence-review
Password: prismm2026

The sequence is in the Awaiting Review tab. You can leave feedback
directly from the sequence page via the Leave Feedback button (opens
a GitHub thread), or just reply to this email.

Keyona
```

---

## Rejection log format (rejections.json)

```json
{
  "rejections": [
    {
      "sequenceId": "seq-NNN",
      "sequenceTitle": "...",
      "rejectedDate": "...",
      "reason": "Martha's specific feedback",
      "pattern": "Short tag — e.g. too-aggressive-opener, vague-subject-lines"
    }
  ]
}
```
