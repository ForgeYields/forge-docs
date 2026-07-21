# Layer 2 — Assets

Hallmark scores every asset a strategy holds or is exposed to — USDC, wstETH, sUSDe, PT tokens, and everything else that can sit in a position — as its own layer: **Layer 2**, the **Asset Risk Score (ARS, 1–10)**. An asset is scored on what it *ultimately* is, not what it is wrapped as: assets that wrap or derive from other assets are assessed on their **ultimate** backing after look-through — a wrapper hop never launders the risk of what sits underneath. Yield-bearing stablecoins, LSTs/LRTs, RWA tokens, and wrappers additionally require a [Layer 1 assessment](layer-1-protocols.md) of their issuing protocol.

> **Canonical rubric.** Criterion weights, scoring bands, and formulas live in the versioned [Layer 2 methodology file](https://github.com/ForgeYields/forge-hallmark/blob/main/methodology/layer2_asset_assessment_methodology.md) — this page describes the structure only. If the two ever diverge, the canonical file wins.

## The ARS rubric (A1–A5)

Each asset is scored on five A-criteria:

| Criterion | What it measures |
|---|---|
| **A1** Peg mechanism | Type, robustness, and mutability of the peg/value mechanism |
| **A2** Depeg history | Historical deviation magnitude, duration, frequency |
| **A3** Liquidity depth | DEX/CEX depth at institutional size, redemption-path reality, exit constraints |
| **A4** Collateral backing | Reserve composition after look-through, attestation quality, custody model, backing counterparties |
| **A5** Market cap / supply concentration | Size and holder concentration, with a bounded structural-holder carve-out |

Alongside the ARS, Layer 2 assessments feed most of the asset-level [classification labels](methodology.md#classification-labels) — `rwa_class`, `tranche_position`, `endogenous_backing`, `redemption_terms_mutable`, `regime_dependent_yield`, `custody_model`, and `backing_counterparties` — the facts a consumer policy keys its cap tightenings and cadence rules to.

## How ARS connects to strategy GRS

The ARS is the asset-risk component of every strategy's [GRS](methodology.md#the-four-layers). Multi-asset strategies are penalized for compositional complexity — the exact composition formula is stated in the [Layer 3 methodology file](https://github.com/ForgeYields/forge-hallmark/blob/main/methodology/layer3_strategy_assessment_methodology.md). Any ARS change propagates to every strategy that depends on the asset.

## Re-scoring cadence

Assets are re-evaluated on the cadence set by the [Allocator Policy](allocator-policy.md) (§6):

- **Monthly** baseline (full A1–A5 re-score), with a narrow blue-chip carve-out running quarterly while attestations stay current; RWA cadence follows the asset's class
- **Event-driven** on material incidents — sustained peg deviations, missed or stale attestations, custodian changes, sharp liquidity drops, changes in backing-counterparty composition

What an ARS *means* for deployment — verdicts, caps, exclusions — is decided by the [Allocator Policy](allocator-policy.md), not here. See [Transparency & public scores](transparency.md) for how to verify any ARS yourself from the public feed at `forge-hallmark/scores/assets/*.yaml`.
