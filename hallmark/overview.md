# 🛡️ Overview

**Hallmark is the open risk-scoring protocol behind every ForgeYields allocation.**

Every strategy deployed in a ForgeYields vault — and every protocol, asset, and chain it touches — is scored by Hallmark before capital can flow to it. Scores are not opinions: they are the output of a published, versioned methodology, applied line by line to source data and committed to a public scoring feed.

Hallmark does one job and does it in the open: it **measures risk**. What to *do* about a given level of risk — deploy, cap, or exclude — is a separate, equally public document: the [ForgeYields Allocator Policy](allocator-policy.md). Keeping the two apart is deliberate: a measurement protocol anyone can adopt, and a decision policy each allocator writes for its own risk appetite.

## Why it exists

ForgeYields targets the yield frontier — strategies most aggregators avoid because they sit outside the blue-chip safe-list. Higher yields demand higher craft, and Hallmark is what that craft looks like in practice: a quantitative measurement of every layer of risk a position touches, before a single dollar moves. Scoring answers five questions before any deposit is made:

1. Is the **chain** itself sound? (consensus, sequencer, bridges, operational track record)
2. Is the underlying **protocol** credible? (audits, governance, incident history, smart-contract risk)
3. Is the **asset** sound? (backing, peg history, liquidity, redemption mechanism)
4. Is the **strategy** itself executable safely? (looping ratios, LP composition, unwind liquidity, oracle dependence)
5. If a third-party vault is in the loop, is the **wrapper** trustworthy? (curator/atomist powers, exit mechanism, fee structure)

Every answer must be defensible to an institutional LP — the way a structured-credit desk defends a tranche. Hallmark is what makes that defense reproducible.

## What it produces

Hallmark scores four layers, each on a 1–10 scale (1 = lowest risk), plus classification labels:

| Layer | Score | Measures |
|---|---|---|
| **L0 — Chain** | CRS | Consensus, sequencer, bridge, and operational risk of the chain itself |
| **L1 — Protocol** | PRS | Systemic risk of each protocol dependency (+ custody-tier labels) |
| **L2 — Asset** | ARS | Backing, peg, and liquidity risk of each asset (+ classification labels) |
| **L3 — Strategy** | GRS | Composite of protocol, asset, and strategy-specific risk for the full position |

Alongside the composites, Hallmark publishes every sub-score and classification label, so any reviewer can re-derive a score from the same public data. The exact rubrics, weights, and composition formulas — and the current methodology version — live in the [Methodology](methodology.md) and in the canonical files on the [public registry](https://github.com/ForgeYields/forge-hallmark).

What Hallmark deliberately does **not** publish: verdicts, eligibility thresholds, position caps, exit timelines, or rebalancing rules. A score of 6.2 is a measurement; whether 6.2 is deployable — and at what size — is an allocator's call.

## The decision layer: the Allocator Policy

ForgeYields makes that call through its own published rulebook, the [**ForgeYields Allocator Policy**](allocator-policy.md). The Policy consumes Hallmark scores and labels and derives:

- **Binary verdicts** — every strategy, protocol, and asset is either APPROVED or EXCLUDED. There is no intermediate verdict.
- **Cap-bands** — allocation ceilings that tighten as scores rise. For example, positions whose scores fall in the upper approved bands (such as the 6.0–6.5 asset band) carry a tighter allocation cap and monthly re-scoring. That band is a *sizing and monitoring rule*, not a verdict or a warning label.
- **Concentration ceilings** — venue, cluster, and regime limits that bind independently of scores.
- **Exit and cascade rules** — how an exclusion anywhere in the dependency tree flows to every strategy that touches it.
- **Reassessment cadence** — how often each layer is re-scored, and which material events trigger an immediate re-score.

Because the split is real, the same Hallmark scores would produce **different deployment decisions at a more or less conservative allocator — by design**. Any institution can adopt Hallmark's scores and write its own policy; ForgeYields' Policy is the public reference implementation.

## How it connects to allocation

```
Hallmark (scores + labels)  →  Allocator Policy (verdicts + caps)  →  fyUSDC / fyETH / fyWBTC vaults
```

- Hallmark owns the canonical scores (YAML, versioned, signed off by the Risk Analyst).
- The Allocator Policy re-derives verdicts and caps from the live scores at allocation time; excluded strategies get zero weight, approved strategies are sized within their cap-bands.
- Score changes (re-scores after incidents, audit updates, governance changes) propagate through the Policy's cascade rules to the next rebalance.
- Reassessment cadence per layer — and the material-event triggers that force an immediate re-score — are defined in the Policy; every re-score is timestamped and the previous version retained for audit.

This is ForgeYields' two-pillar transparency model: **decide before you deposit** with Hallmark's scores and the Allocator Policy — both public, both verifiable — and **audit after** with the [Atomic Transparency Ledger](../basics/atomic-transparency-ledger.md), which records every capital movement together with the policy rationale behind it. For any historical position, you can pull the score that measured it, the policy rule that authorized it, and the ledger entry that executed it.

## Where to go next

- [Methodology →](methodology.md) — the four-layer scoring framework in detail
- [Allocator Policy →](allocator-policy.md) — verdicts, cap-bands, exit rules, cadence
- [How a score is built →](example-score.md) — one strategy, every number
- [Transparency →](transparency.md) — public scores feed, validation, and audit trail
