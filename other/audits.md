# Audits

ForgeYields stands on a three-layer audit stack: the contracts we own, the allocator we license, and the underwriting methodology we publish. All three are independently reviewable.

***

## 1. Clearing Engine (ForgeYields contracts) — [Audited](https://github.com/ForgeYields/audits/blob/main/Forge%20-%20Csc%20Audit%20Report.pdf)

**Scope:** deposits, withdrawals, batching & netting, cross-chain settlement.

**Auditor:** Cairo Security Clan (CSC)\
**Report:** [Csc Audit Report](https://github.com/ForgeYields/audits/blob/main/Forge%20-%20Csc%20Audit%20Report.pdf)

The Clearing Engine has an intentionally minimal attack surface: only deposit and redemption-request functions are exposed, and redemption is asynchronous. Direct liquidity extraction from a single chain is impossible by design, materially limiting exploit blast radius.

***

## 2. Yield Allocator (EVM) — [Audited](https://docs.veda.tech/audits)

**Scope:** strategy execution on EVM chains via Merkle-validated allowed calls.

**Source:** ForgeYields uses the [Veda Labs BoringVault](https://docs.veda.tech) allocator pattern.\
**Audits:** see [Veda's audit page](https://docs.veda.tech/audits) for the full audit chain.

The allocator enforces Merkle-validated allowed calls — the relayer can only interact with approved protocols, methods, and parameters. Off-list calls revert on-chain. This eliminates the "rogue strategist" failure mode that has historically affected other yield aggregators.

***

## 3. Underwriting Methodology ([Hallmark](../hallmark/overview.md))

The Hallmark methodology that gates strategy eligibility is published, versioned, and independently verifiable.

**What's public:**
- The full [rubric](../hallmark/methodology.md) — every criterion, weight, and scoring band
- Every [score](https://github.com/ForgeYields/forge-hallmark) — one YAML per protocol / asset / strategy / chain, with evidence
- Every [amendment](https://github.com/ForgeYields/forge-hallmark/tree/main/methodology/amendments) — dated changes with rationale and backwards-compatibility notes
- All [validation scripts](https://github.com/ForgeYields/forge-hallmark) — `validate-scores.js`, `check-cascade-integrity.js`, `check-drift.js`

**Why this matters:** the audits of our contracts and the allocator prove the *execution* is correct. Hallmark proves the *strategy selection* is defensible. Both are required.

See [Transparency & Public Scores](../hallmark/transparency.md) for how to verify any score yourself.

***

## Underlying-protocol audit chain

Strategies deployed by ForgeYields depend on underlying protocols (Aave, Morpho, Curve, Pendle, Ipor Fusion, MetaMorpho, Convex, Yearn V3, and others). Each carries its own audit chain — typically Spearbit, OpenZeppelin, ChainSecurity, Trail of Bits, Cantina, or equivalent. These are captured in the **C1 (Audit Status)** criterion of each protocol's [Hallmark Layer 1 assessment](../hallmark/methodology.md#layer-1--protocol-risk).

***

## Bug bounty

If you discover a vulnerability in the Clearing Engine, please disclose responsibly to [forge.fi.contact@gmail.com](mailto:forge.fi.contact@gmail.com). A formal bug bounty program is in preparation.
