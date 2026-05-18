---
description: Real performance, full transparency.
---

# 🔑 Yield Allocation

#### **Principles of Allocation**

* **Diversified spread** → Allocate across multiple low-risk, high-APR venues, often incentive-driven or capacity-limited, to maximize risk-adjusted returns.
* **Non-custodial control** → Relayers execute, but every call is verified on-chain against an allow-listed set of protocols, powered by merkle tree based verification system, developed by Veda Labs.&#x20;
* **Automated →** ForgeYields Allocation engine optimizes continuously based on each strategy’s risk-reward profile and operational costs, dynamically managing liquidity distribution and deposit flow without manual intervention

#### **Real-time transparency on every allocation step.**

ForgeYields executes cross-chain allocations automatically and every action becomes instantly visible. Atomic Reports show the allocator’s _true live state_ immediately after each execution.

* **AUM breakdown** by chain and by protocol.
* **Exact executed transactions** with links to explorers.
* **Flow deltas and PnL:** swaps, bridge costs, incentive capture impact.
* **Cash vs positions:** updated balances after every step.
* **Dust tracking:** leftover tokens, residual balances, rounding effects.
* **Recent Transactions feed:** a live stream of the latest actions taken by the allocation engine.
* Exportable **JSON** for integrations, dashboards, or internal monitoring.

#### Risk Management in Operations

Operational risk comes from **loss-potential actions** such as swaps, bridges, or deposits into strategies with entry fees. ForgeYields reduces this risk through strict routing rules, slippage limits, retries, and controlled bridge selection.

**Swap execution risk**\
All swaps (e.g., WETH → wstETH) use KyberSwap with on-chain slippage limits. If pricing exceeds bounds, execution stops and retries later — avoiding bad fills.

**Bridge execution risk**\
Allocations default to canonical bridges.\
In rare cases, when a high-value opportunity requires faster settlement, a reputable non-canonical bridge may be used.

