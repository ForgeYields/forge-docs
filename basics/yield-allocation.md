---
description: How capital flows from deposit to deployed strategy.
---

# 🔑 Yield Allocation


<figure><img src="../.gitbook/assets/rebalancing-pipeline.svg" alt="ForgeYields rebalancing pipeline — inputs (Hallmark scores synced every 15 minutes, Allocator Policy verdicts and limits, epoch net flows from protocol clearing, live monitoring) feed target allocations where excluded strategies get zero weight and all weights respect the caps; the risk committee approves every target; execution runs through crash-safe batch construction and on-chain execution; every movement is recorded in the Atomic Transparency Ledger with its policy rationale, which loops back into continuous monitoring."><figcaption>The allocation pipeline: measure, decide, approve, execute, record — with net inflows sitting as idle and net outflows forming a redemption sleeve, both treated as ordinary allocation targets.</figcaption></figure>
ForgeYields runs a continuous allocator across chains and protocols. Every allocation is gated twice before it executes: [Hallmark](../hallmark/overview.md) scores measure the risk, and the public [Allocator Policy](../hallmark/allocator-policy.md) turns those scores into verdicts and caps. No score, no verdict; no verdict, no capital.

#### **Principles of allocation**

* **Score-gated, policy-bound.** The allocator's strategy universe is the set of strategies APPROVED under the Allocator Policy. Excluded strategies get zero weight — including strategies excluded by cascade from a dependency. Approved strategies are sized within the Policy's cap-bands and concentration ceilings.
* **Diversified across the approved set.** Spread capital across multiple approved venues to maximize risk-adjusted return — not maximize APR, not minimize risk in isolation. The Policy's venue and cluster ceilings make diversification a binding rule, not a preference.
* **Non-custodial.** Vault funds live in on-chain vault contracts (built on [Veda Labs](https://docs.veda.tech) BoringVault infrastructure); depositors hold fyTokens, and ForgeYields never takes custody of user assets.
* **Deliberate.** Every target allocation is reviewed and approved by the risk committee before execution — see the pipeline below.

#### **The allocation pipeline**

Each rebalance runs the same pipeline from inputs to published record:

**1 — Inputs.** The allocator works from: the live Hallmark scores (synced continuously from the public feed), the Allocator Policy's limits (verdicts, cap-bands, ceilings), the epoch's net flows, and live security monitoring of deployed positions. Net flows are ordinary allocation targets like any other: new deposits sit as idle balance until allocated; pending withdrawals form a redemption sleeve that the rebalance funds first.

**2 — Target allocation.** From those inputs the allocator derives a target: excluded strategies at zero, every weight within its Policy cap-band, vault-level ceilings respected. The target is a full picture of where the vault should be, not a list of trades.

**3 — Risk-committee approval.** Every target allocation is approved by the risk committee before anything executes. This is a feature, not a bottleneck — it's the same pattern rating agencies run: *the scorecard indicates, the committee decides.* Scores and policy rules do the quantitative work and constrain the option space; a human sign-off catches what no rulebook anticipates, and the approval is itself part of the audit trail.

**4 — Crash-safe batch construction.** The approved target is decomposed into ordered transaction batches designed so that an interruption at any point leaves the vault in a consistent, recoverable state — no half-executed migration can strand funds.

**5 — On-chain execution.** Relayers execute the batches on-chain, position by position, with each transaction publicly visible on its chain's explorer.

**6 — Published record.** Every movement lands in the **[📜 Atomic Transparency Ledger](atomic-transparency-ledger.md)** together with the policy rationale behind it — which rule sized it, which verdict allowed it. AUM breakdown by chain and protocol, executed transactions with explorer links, flow deltas and PnL, exportable JSON/CSV. New report after every action, no smoothing, no aggregation.

The Ledger is the operational counterpart to the scoring stack: Hallmark and the Policy let you *decide before you deposit*; the Ledger lets you *audit after*.

#### **Risk management in operations**

Operational risk comes from loss-potential actions — swaps, bridges, deposits into strategies with entry fees. ForgeYields mitigates these through strict routing rules.

**Swap execution risk**\
All swaps (e.g., WETH → wstETH) are priced against slippage bounds before execution. If pricing exceeds bounds, execution stops and retries later — avoiding bad fills.

**Bridge execution risk**\
Allocations default to canonical bridges. In rare cases — when a high-value opportunity requires faster settlement — a reputable non-canonical bridge may be used. Bridge risk is captured under Hallmark's [Protocol Risk C5](../hallmark/methodology.md#layer-1--protocol-risk) and [Asset Risk A4](../hallmark/methodology.md#layer-2--asset-risk).

**Allocation drift**\
If a re-score flips a deployed strategy's Policy verdict to EXCLUDED, the Policy's exit rules take over, with timelines scaled to the urgency of the trigger. If a re-score moves a position into a different cap-band, the next rebalance resizes it. See the [Allocator Policy](../hallmark/allocator-policy.md) for the exit and cascade rules.
