# ⚠️ Risks

ForgeYields targets higher-yield strategies than blue-chip aggregators — which means every risk below is real, present, and the price of admission to the yield frontier. Each one is actively measured by **[Hallmark](../hallmark/overview.md)**, the scoring methodology behind every allocation; the ForgeYields [Allocator Policy](../hallmark/allocator-policy.md) derives verdicts from those scores. Strategies without an APPROVED verdict never reach a vault.

## Smart-contract risk

Even audited contracts may contain bugs. The attack surface is intentionally minimal: contracts only expose deposit and redemption requests, and redemptions are asynchronous. This design makes direct liquidity extraction from a single chain impossible, limiting the blast radius of potential exploits.

## Protocol risk

Vulnerabilities or governance compromise in the underlying protocols a strategy uses (Curve, Convex, Aave, Morpho, Pendle, LST/LRT issuers, lending markets, etc.). Each protocol is scored on six weighted criteria — audits, TVL, governance, incident history, smart-contract complexity, team transparency — and rescored on material events. See [Hallmark Methodology → Layer 1](../hallmark/methodology.md#layer-1--protocol-risk).

## Asset risk

Peg failure, backing degradation, or redemption-mechanism breakage in stablecoins, LSTs, LRTs, or wrapped assets. Scored under Hallmark's Layer 2 rubric, including async exit credit for assets with bounded redemption queues and published labels for endogenous backing, regime-dependent yield, and RWA backing class. See [Layer 2 →](../hallmark/methodology.md#layer-2--asset-risk).

## Strategy risk

Execution-level risk: looping leverage ratios, LP composition shifts, oracle dependence, unwind liquidity. Composed with protocol and asset risk into the Global Risk Score (GRS). See [Layer 3 →](../hallmark/methodology.md#layer-3--strategy-risk).

## Wrapper-vault risk

When ForgeYields deposits into a third-party permissioned vault (Ipor Fusion, MetaMorpho, Yearn V3), additional trust assumptions apply: who can move funds, how depositors exit, what fees can change. These are scored under Hallmark's [Type W rubric](../hallmark/methodology.md).

## Liquidity risk

During high demand or volatile periods, redemptions may take longer until liquidity is freed during the next rebalance. The async redemption design ensures fairness across redeemers but does not guarantee instant exit.

## Bridge risk

Even though ForgeYields defaults to canonical bridges, all cross-chain operations carry technical and operational execution risk. Bridge risk is captured within Protocol Risk (C5 Smart Contract Risk) and Asset Risk (A4 Collateral Backing). CCTP and LayerZero DVN are assessed via their underlying validator sets and attestation mechanisms.

## Chain risk

Layer-1 and Layer-2 networks themselves carry consensus, sequencer, and bridge risks. Hallmark scores chains separately from protocols using the **Chain Risk Score (CRS)**; per-chain protocol re-deployments inherit chain risk without being treated as new protocols.

## Concentration risk

Correlated exposure across venues, shared-dependency clusters, and yield regimes. Handled by the [Allocator Policy](../hallmark/allocator-policy.md)'s concentration caps — per-venue, cluster, and regime-dependent-yield aggregate limits — rather than by scoring alone.

---

**Want to verify a specific strategy's risk profile?** Every active strategy has a published Hallmark score and assessment. See [Transparency & Public Scores](../hallmark/transparency.md) for how to pull and verify them.
