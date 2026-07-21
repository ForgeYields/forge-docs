# Layer 0 — Chains

Hallmark scores every chain ForgeYields deploys to — Ethereum, Starknet, Monad, and the rest of the rated registry — as its own layer: **Layer 0**, the **Chain Risk Score (CRS, 1–10)**. A chain is scored before any protocol on it is allowed, and per-chain protocol re-deployments inherit chain risk rather than being treated as new protocols. Chain risk also feeds each strategy's GRS through Protocol Risk via the Multi-Protocol Rule — a strategy on Morpho-Ethereum and a strategy on Morpho-Monad are scored differently because the underlying chains carry different risk.

> **Canonical rubric.** Criterion weights, scoring bands, and formulas live in the versioned [Layer 0 methodology](https://github.com/ForgeYields/forge-hallmark/blob/main/methodology/full-framework.md) — this page describes the structure only. If the two ever diverge, the canonical file wins.

<figure><img src="../.gitbook/assets/chain-registry.svg" alt="Hallmark chain registry — six chains scored under Layer 0 (CRS): Ethereum (L1), Starknet (L2 ZK rollup), Base (L2 OP rollup), Arbitrum (L2 OP rollup), Monad (L1 parallel EVM), HyperEVM (L1 HL EVM). Bands: green low risk (0-5.5), amber elevated (5.5-7.5), red high risk (7.5-10)."><figcaption>Hallmark-rated chains and their Chain Risk Scores (CRS), per the current methodology version. Values shown are a snapshot — see the live registry for current scores.</figcaption></figure>

## The CRS rubric (N1–N5)

Each chain is scored on five N-criteria:

| # | Criterion | What it measures |
|---|---|---|
| **N1** | Consensus & Decentralization | Validator count, client diversity, finality mechanism, super-majority risk |
| **N2** | Sequencer Architecture | L1 (no sequencer) vs L2 sequencer model — centralized, decentralized, escape hatches |
| **N3** | Operational Track Record | Years live, sustained chain halts, recovery patterns |
| **N4** | Bridge Architecture | Native canonical bridge security model for cross-chain operations |
| **N5** | VM Maturity | Years of production code, formal semantics, audit chain on VM upgrades |

## How CRS connects to strategy GRS

Chain risk is scored as its own layer (Layer 0), but it doesn't get a separate slot in the GRS formula. Instead, the CRS **feeds into Protocol Risk** via the Multi-Protocol Rule (MPR):

```
ProtocolRisk = max(P_i) + (N-1) × 0.20 × mean(P_i)
```

Where the chain enters as one of the `P_i` values. This means:
- Strategies on chains with low CRS (e.g. Ethereum) carry minimal chain-risk contribution
- Strategies on chains with elevated CRS (e.g. Monad) inherit measurable additional risk
- Multi-protocol strategies on multiple chains compound both protocol AND chain dimensions

## Re-scoring cadence

Chains are re-evaluated on the cadence set by the [Allocator Policy](allocator-policy.md) (§6):
- **Quarterly** baseline (full N1–N5 re-score)
- **Event-driven** on material incidents — chain halt, sequencer failure, bridge exploit, governance change

See [Transparency & public scores](transparency.md) for how to verify any CRS yourself from the public feed at `forge-hallmark/scores/chains/*.yaml`.
