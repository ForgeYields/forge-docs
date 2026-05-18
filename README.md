# forge-docs

Source of truth for ForgeYields public documentation, synced to **[forge-labs.gitbook.io/forge-docs](https://forge-labs.gitbook.io/forge-docs)** via GitBook Git Sync.

> **Higher yields demand higher craft.**
> Cross-chain yield frontier, underwritten like structured credit.

## What's in this repo

| Path | Purpose |
|---|---|
| `SUMMARY.md` | Sidebar / table of contents for GitBook |
| `hallmark/` | Hallmark — the underwriting framework (Overview, Methodology, Transparency) |
| `basics-risks.md` | Risks page (cross-linked to Hallmark) |
| Other `.md` files | Public docs pages (Basics, How it works, Integration, User guide, etc.) |

## How edits flow to the live docs

```
edit *.md in this repo  →  git push  →  GitBook auto-mirrors
```

GitBook Git Sync watches `main`. Any commit that modifies a page or `SUMMARY.md` triggers a sync to the live site within seconds.

## Adding a page

1. Create `your-section/your-page.md`
2. Add it to `SUMMARY.md` under the appropriate section
3. Commit + push

GitBook respects the `SUMMARY.md` ordering and hierarchy.

## Linking conventions

Use relative markdown links between pages:

```markdown
See [Hallmark Methodology](../hallmark/methodology.md#layer-1--protocol-risk).
```

GitBook rewrites these to its internal page IDs on sync.

## Hallmark methodology

The Hallmark scoring framework is documented in detail in `hallmark/`:
- [Overview](hallmark/overview.md) — what Hallmark is and why it exists
- [Methodology](hallmark/methodology.md) — the three-layer rubric (L1 protocol, L2 asset, L3 strategy + chain risk + wrapper vaults)
- [Transparency](hallmark/transparency.md) — public scores feed, validation, audit trail

Source data lives in [`forge-hallmark`](https://github.com/ForgeYields/forge-hallmark) (public scores) and `forge-hallmark-private` (raw assessments).

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for drafting conventions, information architecture decisions, and integration notes.

## License

Documentation is published under CC BY 4.0. Code samples within docs are MIT unless otherwise noted.
