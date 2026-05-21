# ❓ FAQ

Quick answers to the questions institutional LPs and integrators ask most. For the full methodology see [Methodology](methodology.md); for the public score feed see [Transparency](transparency.md).

***

## The basics

### What is Hallmark, in one sentence?

The underwriting framework that scores every protocol, asset, chain, and strategy ForgeYields touches — published, versioned, and binding on allocation.

### What's the eligibility cutoff?

**GRS ≤ 7.5.** Anything above is excluded. There is no manual override path that bypasses this.

### Who uses Hallmark?

Three audiences:
1. **ForgeYields' allocator** — the binding gate on every deployment
2. **Integrators and LPs** — to verify what underwrites the yield they're holding
3. **The DeFi community** — anyone can pull the public score feed and use it for their own risk decisions

### Is Hallmark open-source?

The methodology, scores, schemas, and validators are all public:
- **Scores:** [github.com/ForgeYields/forge-hallmark](https://github.com/ForgeYields/forge-hallmark)
- **Methodology:** `/methodology/` (current v4.1, with amendments timeline in `/methodology/amendments/`)
- **Validators:** `scripts/validate-scores.js`, `check-cascade-integrity.js`, `check-drift.js`

The raw assessment notes (qualitative analysis behind each score) are kept private under NDA where they reference confidential security disclosures. Public scores are reconstructible from public information alone — private inputs can only *tighten* a score, never loosen it.

***

## How scoring works

### Why these specific layer weights (PR×0.35, AR×0.25, SSR×0.40)?

- **PR weight 0.35** — protocol failure is the most common loss vector empirically (exploit, governance compromise, oracle break). Most weight, but not dominant — because asset and execution risk can independently sink a position.
- **AR weight 0.25** — asset failure (depeg, backing degradation) is real but typically slower-moving and partially mitigated by Hallmark's WATCHLIST band.
- **SSR weight 0.40** — strategy-specific risk is highest weight because it captures *what you're actually doing*. A great protocol holding a great asset in a stupid strategy is still a bad position.

These weights were not chosen via backtesting (which would overfit). They reflect operational priors from observing 18+ months of DeFi incidents and 200+ strategy assessments.

### What's the WATCHLIST band (6.0–6.5)?

Strategies with GRS in **6.0–6.5** are eligible but flagged. They get:
- Capped position sizing (max 15% per vault)
- Monthly re-scoring (vs quarterly baseline)
- Documented monitoring triggers

Above 6.5 the strategy still passes (until 7.5), but allocator preference shifts away from it. Above 7.5 it's excluded.

### What's "cascade integrity"?

A strategy's GRS depends on the protocols and assets it uses. If a protocol gets rescored (e.g., an exploit on Aave), every strategy that uses Aave needs to be re-evaluated. The `check-cascade-integrity.js` validator enforces this: an L1 rescore commit must be accompanied by L3 rescores for every dependent strategy. No orphaned scores allowed.

***

## Cadence and changes

### How often are scores updated?

| Layer | Baseline cadence | Event-driven |
|---|---|---|
| **L1 Protocol** | Quarterly full re-evaluation | Immediate on exploit, governance change, major upgrade |
| **L2 Asset** | Quarterly full re-evaluation; A2/A3/A5 monthly | Immediate on depeg, attestation miss, backing change |
| **L3 Strategy** | On any L1/L2 change in dependency tree (cascade) | Immediate on strategy-specific events |
| **L0 Chain** | Quarterly | On chain-level incident (sequencer fail, bridge exploit) |

The `check-drift.js` validator flags scores older than the cadence threshold.

### What happens if a score changes after I deposit?

If the new score is still ≤ 7.5: the allocator may reweight on the next rebalance, but your position stays.

If the new score crosses above 7.5: the allocator initiates exit from that strategy on the next rebalance. Your fyToken share price reflects the unwound position.

Score changes are visible in real-time in the public feed and in the [Atomic Reports](../basics/yield-allocation.md).

### Can I propose a score change?

Yes. Open an issue or PR on [forge-hallmark](https://github.com/ForgeYields/forge-hallmark) with evidence. Score changes require:
1. Documented evidence (audit report, on-chain incident, governance vote, etc.)
2. Methodology compliance (which criterion(s) are affected, why)
3. Cascade implications (which dependent L3 strategies need rescoring)

The Risk Analyst reviews and merges. Methodology changes (vs individual scores) require a new amendment.

***

## Specific concepts

### What's a Type W wrapper vault?

A strategy that delegates execution to a third-party permissioned vault (Ipor Fusion, MetaMorpho, Yearn V3, generic ERC-4626 wrappers).

Type W exists because the trust model is fundamentally different from direct execution: you're not just trusting Aave's contracts, you're also trusting the curator/atomist who decides what the wrapper does. Hallmark scores this with the X1–X5 rubric (introduced in [v4.1](https://github.com/ForgeYields/forge-hallmark/blob/main/methodology/amendments/v41-amendment.md)):

| Criterion | Weight | What it measures |
|---|---|---|
| **X1** Underlying Strategy Risk | 25% | What the wrapper does economically |
| **X2** Curator/Atomist Trust | 30% | Who can move funds (EOA / MPC / multisig + timelock) |
| **X3** Exit Mechanism | 20% | Atomic / queue / pause history |
| **X4** Fee Structure | 15% | Transparent vs unilateral change |
| **X5** Vault Maturity | 10% | Track record, TVL, incidents |

X2 carries 30% because curator trust is the defining differentiator between a robust wrapper and a single-EOA-can-rug situation.

### What's CRS (Chain Risk Score)?

Layer-0 chain risk, introduced in [v4.0](https://github.com/ForgeYields/forge-hallmark/blob/main/methodology/amendments/v40-amendment.md). Scores chains independently using the N1–N5 rubric (consensus, sequencer, operational track record, bridge, VM maturity). Chain risk feeds into protocol risk via the existing Multi-Protocol Rule — no new weight constants.

This closes the gap where the same Morpho deployment on Ethereum vs Monad was scored identically pre-v4.0.

### What's VRS (Vault Risk Score)?

Vault-level risk: the **allocation-weighted average of strategy GRS** across a vault's current positions, plus a **coverage** percentage (fraction of allocation with current Hallmark scores).

Where GRS is per-strategy, VRS is per-vault. fyUSDC, fyETH, and fyWBTC each have a current VRS visible in the app.

***

## Verification

### How do I verify a score myself?

1. Pick a strategy from the [Strategies API](../integration/api/strategies-info.md) or the fyToken page
2. Find its YAML in the public Hallmark feed at `forge-hallmark/scores/strategies/<slug>.yaml`
3. Cross-check the `criteria` block against the methodology version it references
4. Read the assessment markdown for qualitative reasoning
5. If you find a discrepancy, raise an issue

If you want a worked example: see [How a score is built — gteUSDc Morpho](example-score.md).

### Is Hallmark itself audited?

Hallmark is a methodology, not a smart contract — it's verified by its inputs and outputs being public, not by a code audit. The validators (`validate-scores.js`, `check-cascade-integrity.js`, `check-drift.js`) are open-source and run on every change. Anyone can fork the methodology and apply it independently to verify our scores.

The contracts that *enforce* Hallmark eligibility (the allocator's allow-list mechanism) are audited as part of the Clearing Engine + Veda BoringVault — see [Audits](../other/audits.md).

***

## For partners

### Can I plug Hallmark scores into my own product?

Yes. The score feed is freely readable from the public repo, or pull through the ForgeYields API (a dedicated Hallmark API endpoint is on the roadmap). Attribution requested but not required.

### How does this differ from "audited"?

An audit is a one-time review of code. Hallmark is a continuous, methodology-driven assessment of *operational* and *strategy* risk — most of which is invisible to a code audit.

A protocol can have 5 audits and a Hallmark PR score of 7.5 (e.g., because it relies on off-chain CEX hedging). Conversely, a protocol with fewer audits can score well if its operational profile is otherwise strong. The two are complementary, not substitutes.
