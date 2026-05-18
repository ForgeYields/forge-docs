# GitBook drafts — Hallmark integration

This folder contains drafts for adding **Hallmark** (the risk assessment engine) to the public ForgeYields GitBook at https://forge-labs.gitbook.io/forge-docs.

## What's here

| File | Where it goes on GitBook |
|---|---|
| `hallmark/overview.md` | **New page** — `Hallmark > Overview` |
| `hallmark/methodology.md` | **New page** — `Hallmark > Methodology` |
| `hallmark/transparency.md` | **New page** — `Hallmark > Transparency & Public Scores` |
| `basics-risks.md` | **Replaces** the existing `Basics > Risks` page (adds cross-links to Hallmark) |
| `SUMMARY.md` | Proposed full sidebar with the new `Hallmark` section inserted between `Basics` and `How it works` |

## Information architecture decision

Hallmark gets its **own top-level section** rather than living under `Basics > Risks` because:
- It's a product, not a disclaimer — institutional LPs need to find it without scrolling past user-facing content.
- The methodology page is too long to nest under a Basics page.
- The existing `Risks` page stays as a plain-language summary, with each risk cross-linked to the relevant Hallmark layer.

## How to land these on the live GitBook

Three options, easiest to most automated:

### Option A — Manual paste (fastest, no setup)
1. In GitBook, create a new top-level section **"Hallmark"** between *Basics* and *How it works*.
2. Add three pages (`Overview`, `Methodology`, `Transparency & Public Scores`) and paste in the content from each file.
3. Edit the existing **Basics > Risks** page and replace its body with `basics-risks.md`.
4. Adjust internal links — GitBook will rewrite relative paths to its internal page IDs on save.

### Option B — GitBook GitHub sync (recommended for ongoing updates)
GitBook supports two-way sync with a GitHub repo. If you set this up once, future doc changes flow from this repo to GitBook on push.
1. In GitBook → *Integrations → Git Sync*, connect this repo (or a dedicated docs repo) and point the sync at `gitbook/` as the root.
2. Use the included `SUMMARY.md` as the table-of-contents source.
3. Push the existing pages from GitBook into the repo (one-time export), then add these files alongside them.

### Option C — GitBook API (scriptable)
GitBook has a REST API for creating/updating pages. Needs a personal access token and the space ID (the `JxgiuUCN5gdGZzp4W9Xu` in the GitBook URL). I can write a script that POSTs each draft as a page — say the word and supply a token.

## Quality bar

These drafts are written for the institutional LP reader. They are factually grounded in:
- `methodology/risk_methodology.md` (v3.9 base + v4.0/v4.1 amendments)
- `methodology/v40_amendment.md` (Chain Risk / CRS)
- `methodology/v41_amendment.md` (Type W wrapper vaults)
- `.claude/agents/fy-risk.md` (Hallmark file layout)

Before publishing, sanity-check:
- The methodology version numbers in `methodology.md` match the current state of `methodology/`.
- The cascade integrity / drift validator names match what's actually in `forge-hallmark-private/scripts/`.
- The "What's public" section in `transparency.md` aligns with what you actually intend to publish (it currently assumes the YAML scores feed is publicly readable).
