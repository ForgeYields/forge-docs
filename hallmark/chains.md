# ⛓ Chains

Hallmark scores every chain ForgeYields deploys to before any protocol on it is allowed. Chain risk feeds into Protocol Risk via the Multi-Protocol Rule — a strategy on Morpho-Ethereum and a strategy on Morpho-Monad are scored differently because the underlying chains carry different risk.

<figure><img src="../.gitbook/assets/chain-registry.svg" alt="Hallmark chain registry — six chains scored under Layer 0 (CRS): Ethereum 1.45 (L1), Starknet 5.50 (L2 ZK rollup), Base 3.40 (L2 OP rollup), Arbitrum 3.25 (L2 OP rollup), Monad 5.95 (L1 parallel EVM), HyperEVM 6.35 (L1 HL EVM). Bands: green low risk (0-5.5), amber elevated (5.5-7.5), red excluded (7.5-10)."><figcaption>Current Hallmark-rated chains and their Chain Risk Scores (CRS), per methodology v4.2.</figcaption></figure>

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

Chain risk doesn't get its own slot in the GRS formula. Instead, it **feeds into Protocol Risk** via the Multi-Protocol Rule (MPR):

```
ProtocolRisk = max(P_i) + (N-1) × 0.20 × mean(P_i)
```

Where the chain enters as one of the `P_i` values. This means:
- Strategies on chains with low CRS (Ethereum 1.45) carry minimal chain-risk contribution
- Strategies on chains with elevated CRS (Monad 5.95) inherit measurable additional risk
- Multi-protocol strategies on multiple chains compound both protocol AND chain dimensions

## Re-scoring cadence

Chains are re-evaluated:
- **Quarterly** baseline (full N1–N5 re-score)
- **Event-driven** on material incidents — chain halt, sequencer failure, bridge exploit, governance change

See [Transparency & Public Scores](transparency.md) for how to verify any CRS yourself from the public feed at `forge-hallmark/scores/chains/*.yaml`.
