# Hallmark

**Hallmark is the underwriting framework behind every ForgeYields allocation.**

Every strategy deployed in a ForgeYields vault — and every protocol, asset, and chain it touches — is underwritten by Hallmark before capital can flow to it. Allocations are not opinions: they are the output of a published, versioned methodology, applied line by line to source data and committed to a public scoring feed.

## Why it exists

ForgeYields targets the yield frontier — strategies most aggregators avoid because they sit outside the blue-chip safe-list. Higher yields demand higher craft, and Hallmark is what that craft looks like in practice: a quantitative bar that every strategy must clear across protocol, asset, and execution risk before a single dollar moves. Underwriting answers four questions before any deposit is made:

1. Is the underlying **protocol** credible? (audits, governance, incident history, smart-contract risk)
2. Is the **asset** sound? (backing, peg history, liquidity, redemption mechanism)
3. Is the **strategy** itself executable safely? (looping ratios, LP composition, unwind liquidity, oracle dependence)
4. If a third-party vault is in the loop, is the **wrapper** trustworthy? (curator/atomist powers, exit mechanism, fee structure)

Every "yes" must be defensible to an institutional LP — the way a structured-credit desk defends a tranche. Hallmark is what makes that defense reproducible.

## What it produces

Hallmark emits three numbers per strategy, on a 1–10 scale (1 = lowest risk):

| Score | Meaning | Eligibility |
|---|---|---|
| **Protocol Risk (PR)** | Systemic risk of the protocol | Input to GRS |
| **Asset Risk (AR)** | Backing, peg, and liquidity risk of the asset | Input to GRS |
| **Global Risk Score (GRS)** | Composite: `PR × 0.35 + AR × 0.25 + SSR × 0.40` | **≤ 7.5** to be eligible |

Strategies scoring **> 7.5** are excluded from vaults. Strategies in the **6.0–6.5 WATCHLIST band** require manual review and capped position sizing.

## How it connects to allocation

```
Hallmark (scoring)  →  forge-backend (allocator)  →  fyUSDC / fyETH / fyWBTC vaults
```

- Hallmark owns the canonical scores (YAML, versioned, signed off by the Risk Analyst).
- The allocator consumes scores via a public feed and refuses to deploy capital to any strategy with no current score or a GRS > 7.5.
- Score changes (re-scores after incidents, audit updates, governance changes) propagate to the allocator on next rebalance.

This means allocation decisions are **auditable**: for any historical position, you can pull the Hallmark score that authorized it.

## Cadence

- **Protocol scores (L1):** re-evaluated quarterly or on material events (exploit, governance change, major upgrade).
- **Asset scores (L2):** re-evaluated quarterly or on peg events, backing changes, or redemption mechanism changes.
- **Strategy scores (L3):** re-evaluated on any L1 or L2 change in their dependency tree (cascade), or on strategy-specific events.

Re-scores are timestamped and the previous version is retained for audit.

## Where to go next

- [Methodology →](methodology.md) — the three-layer framework in detail
- [Transparency →](transparency.md) — public scores feed, validation, and audit trail
