---
description: Real-time operational state of every allocator action.
---

# 📜 Atomic Transparency Ledger

**The Atomic Transparency Ledger is the operational counterpart to [Hallmark](../hallmark/overview.md).** Where Hallmark publishes the methodology and per-strategy risk scores (*what's allowed in*), the Ledger publishes the real-time state of every allocator action (*what's actually happening*).

Together they form the two pillars of ForgeYields transparency:

| Pillar | Surface | Question it answers |
|---|---|---|
| 🛡 **Hallmark** | Per-strategy risk scores, methodology, eligibility | *Why is this strategy in the vault?* |
| 📜 **Atomic Transparency Ledger** | Live allocator state, every transaction, AUM, PnL | *What is the vault actually doing right now?* |

Most yield aggregators publish neither. ForgeYields publishes both.

---

## Why "Atomic"

Each report captures **one allocator action in full** — a single rebalance, a single deposit batch, a single redemption settlement. Reports aren't smoothed, aggregated, or interpolated between executions. If the allocator did three things, you see three reports. Each one is an atomic snapshot of the system's true state immediately after that action.

This is deliberate: lossy aggregation hides what actually happened. Atomic reporting preserves the audit trail.

---

## What every Atomic Report contains

Every report published after an allocator execution includes:

| Field | What it tells you |
|---|---|
| **AUM breakdown** | Total assets under management, split by chain *and* by protocol — at the moment of execution |
| **Executed transactions** | Every on-chain call the allocator made in that step, with explorer links — anyone can verify on-chain |
| **Flow deltas and PnL** | Swap slippage, bridge costs, incentive capture — net effect of the execution on vault value |
| **Cash vs positions** | Updated buffer balances vs deployed positions after the step |
| **Dust tracking** | Leftover tokens, residual balances, rounding effects — nothing swept under the rug |
| **Recent transactions feed** | Live stream of the latest allocator actions across the system |
| **Exportable JSON / CSV** | Machine-readable for dashboards, internal monitoring, or institutional reporting |

---

## Refresh cadence

The Ledger publishes a new Atomic Report **after every allocator execution** — typically every 30 minutes during active rebalancing, sometimes more frequently during cross-chain settlement cycles. There is no batched daily summary that hides intra-day moves; the live state is always current to the most recent action.

---

## How it complements Hallmark

| Scenario | Hallmark answers | Atomic Transparency Ledger answers |
|---|---|---|
| "Why is this strategy in my vault?" | Published GRS ≤ 7.5, criterion breakdown, methodology version | — |
| "How much capital is in it right now?" | — | AUM breakdown at the latest Atomic Report |
| "What did the allocator do in the last hour?" | — | Last 1–2 Atomic Reports with executed transactions |
| "What's the post-mortem on this loss?" | Hallmark criterion that triggered the cascade | Atomic Reports during and after the incident showing exact flows |
| "Was this strategy actually used last week?" | — | Historical Atomic Reports show real allocation weights over time |

**Hallmark gates. The Ledger observes.** A claim that a strategy is "Hallmark-rated and currently active" can be verified against both: the published score in the [Hallmark feed](../hallmark/transparency.md), and the live allocation in the most recent Atomic Report.

---

## How to access

* **In the app** — every vault page surfaces its latest Atomic Report and recent transaction stream.
* **Via API** — [`api.forgeyields.com/strategies/strategies-info`](../integration/api/strategies-info.md) returns the current allocator state; per-strategy detail endpoints expose historical Atomic Reports.
* **As JSON/CSV exports** — for institutional partners running internal dashboards or risk monitoring. Contact: [forge.fi.contact@gmail.com](mailto:forge.fi.contact@gmail.com).

---

## What you can verify yourself

A short non-exhaustive list of things institutional readers commonly verify from the Ledger:

* **AUM reconciliation** — sum of Atomic Report AUM segments matches the on-chain vault balance
* **Execution authenticity** — every transaction in the report appears on-chain at the reported hash
* **Slippage tracking** — actual fills vs expected prices from the report's PnL deltas
* **Dust accounting** — residual balances reported match on-chain holdings (no quiet sweeps)
* **Allocation drift response** — when a Hallmark score crosses the eligibility cutoff, the corresponding unwind appears in the next Atomic Report

If you find a discrepancy, raise it — operational transparency only works if it's checked. Contact channel above.

---

## Related

* 🔑 [Yield Allocation](yield-allocation.md) — how capital flows from deposit to deployed strategy (the actions the Ledger logs)
* 🛡 [Hallmark — Overview](../hallmark/overview.md) — the risk methodology that gates which strategies the allocator can touch
* 📂 [Hallmark — Transparency & Public Scores](../hallmark/transparency.md) — the methodology side of ForgeYields' transparency
* 📊 [Performance Tracking](performance-tracking.md) — Projected vs Realized yield, computed from Atomic Report history
