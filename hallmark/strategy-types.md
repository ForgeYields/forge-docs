# 🧩 Strategy types

Hallmark scores 6 strategy types. Each type has its own S-criteria rubric (or X-criteria for Type W), reflecting that the *kinds of things that can go wrong* differ structurally between a looping position, an LP, a lending supply, and a wrapper vault.

The type determines the **SSR (Strategy-Specific Risk)** component of the GRS. PR (Protocol Risk) and AR (Asset Risk) are computed the same way regardless of type.

***

## Overview

| Type | Display name | Rubric | Where capital sits | Example |
|---|---|---|---|---|
| **1** | Looping | S1–S4 | Lending market, leveraged | wstETH/WETH looper on Aave |
| **2A** | LP Classic | S1–S4 | Two-asset AMM pool | wstETH/WETH on Curve |
| **2B** | Pendle LP | S1–S4 | Pendle LP (PT + SY) | LP-sUSDe-13aug2026 |
| **2C** | PT Bond-like | S1–S4 | Pendle PT held to maturity | PT-cUSD-23jul2026 |
| **3** | Lending | S1–S5 | Direct supply, unleveraged | USDC supply on Morpho Blue |
| **W** | Wrapper Vault | X1–X5 (v4.1) | Third-party permissioned vault | gteUSDc Morpho, ipsrBTC Ipor Fusion |

***

## Type 1 — Looping

**What it is:** Deposit collateral → borrow against it → swap borrowed asset back to collateral type → re-deposit. Repeat N times. The position is leveraged exposure to the spread between the collateral's yield and the borrow cost.

**Example:** `morpho-blue-wsteth-weth-looping` — deposit wstETH, borrow WETH, swap WETH → wstETH, redeposit. Captures the stETH yield with leverage.

**Rubric (S-criteria):**

| # | Criterion | What it measures |
|---|---|---|
| **S1** | Leverage-Adjusted Correlation Risk | How correlated are the borrow and collateral assets? wstETH↔WETH is highly correlated; less correlated pairs deserve higher scores. |
| **S2** | Stress Liquidation Buffer | Buffer between current LTV and liquidation LTV under stress (a 10% depeg scenario). Tighter buffer = higher risk. |
| **S3** | Oracle Risk | Does the lending market use a robust oracle for the collateral/borrow price? Pyth/Chronicle vs custom feeds. |
| **S4** | Unwind Liquidity | Can the position be unwound in stressed markets without significant slippage? |

**Common failure modes:**
- Stress liquidation when LST depegs and the lending oracle marks it down
- Forced unwind into thin liquidity (slippage cost exceeds expected yield)
- Borrow rate spike making the spread negative

***

## Type 2A — LP Classic

**What it is:** Two-asset liquidity provision on AMMs like Curve, Balancer, Sushi. Earn swap fees + (often) external incentive rewards (CRV, CVX, etc.).

**Example:** `frxusd-msusd-curve-plusconvex-lp` — provide LP on the frxUSD/msUSD Curve pool, stake the LP token into Convex for boosted rewards.

**Rubric (S-criteria):**

| # | Criterion | What it measures |
|---|---|---|
| **S1** | IL Risk | Impermanent loss exposure. Stable-stable pairs are low; volatile-volatile is high. |
| **S2** | Depeg Correlation | If one asset depegs, what happens to LP value? Pegged pairs may experience asymmetric drainage. |
| **S3** | Dual Contract Surface | Both pool contract + (often) reward gauge contract introduce surface area. |
| **S4** | Reward Token Risk | If incentives are in volatile reward tokens (CRV, CVX), what's the price/illiquidity risk? |

**Common failure modes:**
- One side of the pool depegs and arbitrageurs drain the other side
- Reward token price collapse (e.g., during reward emission cuts)
- Gauge weight migration away from the pool

***

## Type 2B — Pendle LP

**What it is:** Provide LP on Pendle pools (PT + SY token pair). Earns swap fees and pool incentives, while having directional exposure to the yield rate.

**Example:** `pendle-lp-susde-13aug2026` — LP on the sUSDe August 2026 pool.

**Rubric (S-criteria):**

| # | Criterion | What it measures |
|---|---|---|
| **S1** | Yield Divergence Risk | How much can the underlying yield rate diverge from the implied? |
| **S2** | Underlying Asset Risk | Direct exposure to the underlying yield-bearing asset (sUSDe, stETH, etc.). |
| **S3** | Maturity Risk | Position should be unwound or rolled before maturity — gets riskier as time decays. |
| **S4** | Pool Liquidity | Pendle pools are concentrated near maturity, then collapse post-expiry. |

**Common failure modes:**
- Underlying yield rate spikes after entry (LP holders lose to PT/YT traders)
- Pool liquidity dries up before planned exit
- Underlying asset depeg propagates into the LP

***

## Type 2C — PT Bond-like (held to maturity)

**What it is:** Buy a Pendle Principal Token (PT) and hold it to maturity. PT is essentially a zero-coupon bond on the underlying yield asset — locked-in yield, predictable payout.

**Example:** `pendle-pt-cusd-23jul2026` — hold PT-cUSD until July 23, 2026.

**Rubric (S-criteria):**

| # | Criterion | What it measures |
|---|---|---|
| **S1** | Underlying Asset Risk | Counterparty risk on the asset that pays out at maturity. |
| **S2** | Maturity Risk | Time remaining; longer = more exposure to underlying changes. |
| **S3** | Yield Source Quality | Is the yield real (e.g., real-world treasuries) or rebate-style (incentives that could be cut)? |
| **S4** | PT Exit Liquidity | If you need to exit early, can you sell PT at fair value? |

**Common failure modes:**
- Underlying issuer fails before maturity (cUSD depeg, etc.)
- Need to exit early into illiquid PT market at discount
- Yield source revealed to be incentive-rebate rather than organic

***

## Type 3 — Lending

**What it is:** Direct supply of an asset to a lending market. Earn supply APY paid by borrowers. No leverage, no LP token, no maturity.

**Example:** Direct USDC supply into a Morpho Blue isolated market.

**Rubric (S-criteria):**

| # | Criterion | What it measures |
|---|---|---|
| **S1** | Risk Management Quality | Does the market have responsible IR curves, LLTV, oracle config? |
| **S2** | Collateral Composition | What can be used as collateral against your supply? |
| **S3** | Utilization Spike Risk | What happens if utilization → 100%? Can you still withdraw? |
| **S4** | Collateral Correlation Risk | If multiple collateral types correlate (e.g., all LSTs), simultaneous depeg risk. |
| **S5** | Bad Debt History | Has this market or its parent ever accumulated bad debt? |

**Common failure modes:**
- Bad collateral added by misconfigured market deployer
- Utilization stuck at 100% during stress (can't withdraw)
- Bad debt socialized to suppliers

***

## Type W — Wrapper Vault (v4.1)

**What it is:** Deposit into a third-party permissioned vault (Ipor Fusion, MetaMorpho, Yearn V3, generic ERC-4626) that internally executes its own strategy. ForgeYields trusts the wrapper to deploy responsibly — but that trust is *itself* what Hallmark scores.

**Example:** `gteusdc-morpho` (Gauntlet curates a MetaMorpho vault), `taucbeth-ipor-fusion-base` (Ipor Fusion vault on Base).

**Why this type exists:** Pre-v4.1, wrapper vaults were forced into Types 1/2/3 based on what the wrapper *did economically*. That missed the two-layer trust structure entirely. A vault doing cautious lending with a single-EOA atomist is structurally riskier than one doing aggressive looping with a 5/9 multisig + 48h timelock. Type W captures this.

**Rubric (X-criteria, weighted differently from S-criteria):**

| # | Criterion | Weight | What it measures |
|---|---|---|---|
| **X1** | Underlying Strategy Risk | 25% | What the wrapper does economically (looping, LP, lending) — scored as if direct. |
| **X2** | Curator/Atomist Trust | **30%** | Who can move funds. EOA = 9 (hard cap), MPC-attested = 6–8, multisig + timelock = 3–5. |
| **X3** | Exit Mechanism | 20% | Atomic = 1–2, queue ≤ 1d = 3–4, queue ≤ 7d = 5–6, queue > 7d or pause history = 7–9. |
| **X4** | Fee Structure | 15% | Transparent low = 1–3, standard = 4–5, opaque/high = 6–8, unilateral change = 8–9. |
| **X5** | Vault Maturity | 10% | >2y + >$100M = 1–3, >1y + >$10M = 4–5, <1y or <$10M = 6–8, <3 months or recent incident = 8–10. |

**X2 carries the highest weight because curator trust is the defining differentiator.** If the curator is a single EOA that can move all vault funds in one transaction, no other criterion can save the wrapper.

**Common failure modes:**
- Single-EOA atomist rugs (X2 = 9 hard cap usually prevents)
- Curator unilaterally raises fees mid-position (X4 spike)
- Vault TVL collapses below liquidity floor (X5 spike + X3 affected)
- Underlying strategy degrades faster than the wrapper rebalances

***

## How to read a strategy YAML

Each strategy in the public feed at [`forge-hallmark/scores/strategies/`](https://github.com/ForgeYields/forge-hallmark/tree/main/scores/strategies) has:

```yaml
name: gteUSDc Morpho (Gauntlet eUSD Core)
strategy_type: W           # ← the type (1, 2A, 2B, 2C, 3, W)
chain: ethereum
assessment:
  date: 2026-05-18
  methodology_version: v4.1
grs: 3.63                  # ← the composite GRS
components:
  protocol_risk: 2.2       # ← PR
  asset_risk: 4.4          # ← AR
  strategy_specific_risk: 4.40  # ← SSR
strategy_specific_criteria:
  x1: 4                    # ← per-criterion scores (X for Type W, S for others)
  x2: 5
  x3: 4
  x4: 4
  x5: 5
dependencies:
  chains: [ethereum]
  protocols: [morpho-blue]
  assets: [eusd, wbtc, eth-plus, wsteth]
```

The `strategy_type` field tells you which rubric was used. The `strategy_specific_criteria` block lets you inspect every per-criterion score the SSR was computed from.

For a full worked walkthrough, see [How a score is built — gteUSDc Morpho](example-score.md).
