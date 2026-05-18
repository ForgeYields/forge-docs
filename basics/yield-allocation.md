---
description: How capital flows from deposit to deployed strategy.
---

# 🔑 Yield Allocation

ForgeYields runs a continuous allocator across chains and protocols. The allocator is fully automated, but every step is gated by [Hallmark](../hallmark/overview.md) — strategies are only eligible if their published GRS is ≤ 7.5.

#### **Principles of allocation**

* **Hallmark-gated.** The allocator's strategy universe is the set of strategies that pass Hallmark eligibility. No GRS, no allocation. A GRS that drifts above 7.5 triggers immediate unwind, not optimization.
* **Diversified across the eligible set.** Spread capital across multiple eligible venues to maximize risk-adjusted return — not maximize APR, not minimize risk in isolation.
* **Non-custodial execution.** Relayers execute, but every call is verified on-chain against a Merkle-validated allow-list (powered by [Veda Labs](https://docs.veda.tech) BoringVault). Off-allow-list calls revert.
* **Continuous.** The allocator re-optimizes based on each strategy's risk-reward profile, incentive flow, and operational costs. No manual intervention.

#### **Real-time transparency on every allocation step**

ForgeYields publishes Atomic Reports immediately after each execution. Each report shows the allocator's true live state:

* **AUM breakdown** by chain and by protocol
* **Executed transactions** with explorer links
* **Flow deltas and PnL:** swaps, bridge costs, incentive capture
* **Cash vs positions:** updated balances after every step
* **Dust tracking:** leftover tokens, residual balances, rounding effects
* **Recent transactions feed:** live stream of allocator actions
* Exportable **JSON** for integrators, dashboards, monitoring

#### **Risk management in operations**

Operational risk comes from loss-potential actions — swaps, bridges, deposits into strategies with entry fees. ForgeYields mitigates these through strict routing rules.

**Swap execution risk**\
All swaps (e.g., WETH → wstETH) use KyberSwap with on-chain slippage limits. If pricing exceeds bounds, execution stops and retries later — avoiding bad fills.

**Bridge execution risk**\
Allocations default to canonical bridges. In rare cases — when a high-value opportunity requires faster settlement — a reputable non-canonical bridge may be used. Bridge risk is captured under Hallmark's [Protocol Risk C5](../hallmark/methodology.md#layer-1--protocol-risk) and [Asset Risk A4](../hallmark/methodology.md#layer-2--asset-risk).

**Allocation drift**\
If a deployed strategy's Hallmark score is updated and crosses the eligibility cutoff, the allocator initiates exit on the next rebalance. See [Hallmark cadence](../hallmark/overview.md) for how rescores propagate.
