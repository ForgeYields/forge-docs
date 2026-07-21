# 📚 Features

#### **Underwritten by Hallmark**

Every strategy ForgeYields touches is scored against a [published methodology](../hallmark/overview.md) before any capital flows. Protocol, asset, chain, and (for wrapper vaults) curator/atomist trust — all measured on a 1–10 scale, with a hard cutoff at GRS ≤ 7.5. Strategies that don't clear the bar never reach a vault.

This is the defining feature. Everything else builds on it.

#### **Frontier yields, cross-chain**

ForgeYields targets the yields most aggregators avoid — strategies that sit outside the blue-chip safe-list because they require active underwriting to safely deploy. Capital rotates across chains as opportunities open and close, with every venue pre-cleared by Hallmark.

* Higher net returns than blue-chip-only aggregators
* Cross-chain coverage: Ethereum, Starknet, Base, Arbitrum, Monad, HyperEVM
* No manual chasing — the allocator deploys, monitors, and unwinds positions

#### **Cross-rollup liquidity efficiency**

Deposits and withdrawals batch on each rollup and only the net difference settles on Ethereum. For users, this means a chain-agnostic experience:

* No manual bridging
* No gas surprises
* No waiting for cross-chain sync
* Liquidity moves in the background, automatically

#### **DeFi-ready fyTokens**

fyTokens follow the ERC-4626 standard, making them fully composable across DeFi:

* Use as collateral in lending markets
* Integrate into other protocols
* Bridge instantly across L2s

Your yield-bearing position is an asset you can use, not a locked deposit.

#### **Asynchronous redemptions, by design**

Deposits mint instantly. Withdrawals take slightly longer because your capital is actively earning across multiple chains. This is intentional — the async pattern is how the vault limits liquidity-extraction attacks.

* A small redemption fee (visible in the UI, per strategy)
* Timing depends on liquidity availability
* Every redemption tracked in the [Atomic Transparency Ledger](atomic-transparency-ledger.md)

#### **Full transparency**

Every Hallmark score is [public](../hallmark/transparency.md), every methodology amendment is dated and signed, every allocator action is logged in the [Atomic Transparency Ledger](atomic-transparency-ledger.md). If a position later goes wrong, the post-mortem can attribute the failure to a specific criterion — not vague "model failure."

#### **Non-custodial control**

You hold fyTokens. ForgeYields never holds your private keys or your funds. Relayers execute, but every call is verified on-chain against a Merkle-validated allow-list (powered by [Veda Labs](https://docs.veda.tech) BoringVault). No relayer can deviate from the pre-approved set.
