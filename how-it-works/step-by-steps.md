---
description: Understand ForgeYields, step by step
---

# 🔄 Step by Steps

#### 1. Deposit from Any Chain

Start by depositing your crypto into a ForgeYields strategy from any supported chain. Your assets are pooled in a local buffer with other deposits. In return, you instantly mint a yield-bearing token — your **fyToken**.

<figure><img src="../.gitbook/assets/step-1-deposit.svg" alt="Step 1 — User wallet deposits assets to the Token Gateway (Starknet or Ethereum) and receives fyToken shares (ERC-4626, directional in underlying)."><figcaption>User → Token Gateway → fyToken.</figcaption></figure>

#### 2. Netting and Settlement

Deposits and withdrawals are **batched and netted** at the rollup level. Then, all local deltas are globally netted across chains and settled on Ethereum L1. This approach drastically reduces gas usage and avoids unnecessary bridge hops.

<figure><img src="../.gitbook/assets/step-2-netting.svg" alt="Step 2 — Per-chain deposits and withdrawals are batched at the rollup level, then a single net delta is broadcast to the Controller on Starknet."><figcaption>Hundreds of user-level flows → one cross-chain net delta.</figcaption></figure>

#### 3. Ethereum Vault Settlement

Only the final net result is processed through Ethereum's canonical bridge. Assets are either deposited into or withdrawn from the main vault, ensuring all yield activity remains Ethereum-secured.

<figure><img src="../.gitbook/assets/step-3-vault-settlement.svg" alt="Step 3 — The net delta from the Controller flows through Ethereum's canonical bridge into the main Vault (Veda BoringVault on Ethereum)."><figcaption>Only the final net result is bridged. All yield activity Ethereum-secured.</figcaption></figure>

#### 4. Strategy Eligibility (Hallmark)

Before any strategy can receive capital, it must clear [Hallmark](../hallmark/overview.md) — ForgeYields' published underwriting framework. Hallmark scores every protocol, asset, chain, and (for wrapper vaults) curator on a 1–10 scale. The composite **Global Risk Score (GRS)** must be ≤ 7.5 for the strategy to enter the allocator's eligible set. Scores are versioned, publicly verifiable, and rescored on material events.

#### 5. Strategy Execution

The allocation engine deploys across the eligible set. All actions are strictly validated against a Merkle-tree whitelist of allowed calls — the relayer cannot deviate from the pre-approved set. If a strategy's GRS later drifts above 7.5, the allocator unwinds the position on the next rebalance.

<figure><img src="../.gitbook/assets/step-5-execution.svg" alt="Step 5 — The Vault dispatches via the Manager, which enforces a Merkle-validated allow-list. Capital fans out into categorized integrations: lending (Aave, Morpho, Spark), LP/DEX (Curve, Convex, Balancer), yield (Pendle, Yearn V3), and wrappers (Ipor Fusion, MetaMorpho). Off-list calls revert."><figcaption>Vault → Manager → Merkle-validated allow-list → integrations. Off-list calls revert on-chain.</figcaption></figure>

#### 6. Earn Yield Passively

Your **fyToken** grows in value automatically based on the performance of the underlying strategy. Meanwhile, you can freely use the token in DeFi, move it across chains, or explore new ecosystems — all without touching the underlying assets or relying on third-party bridges.

<figure><img src="../.gitbook/assets/step-6-earn.svg" alt="Step 6 — Auto-compounding visualization: fyToken share price grows over time while the token count stays constant."><figcaption>Auto-compounded. Your token count never changes — each token grows in value.</figcaption></figure>

#### 7. Redeem Anytime, from Anywhere

When you're ready, redeem your fyToken on any supported chain. The underlying assets are released once the vault settles liquidity, typically after the next strategy rebalancing.

> 🔁 **Redemptions are asynchronous** — your capital continues earning until the last moment, and the delayed settlement helps reduce exploit risk by limiting direct liquidity exposure.

<figure><img src="../.gitbook/assets/step-7-redeem.svg" alt="Step 7 — User calls requestRedeem on the Token Gateway, which mints an ERC-721 redeem NFT recording the epoch and shares. After the next clearing cycle, a relayer calls claimRedeem and the underlying is delivered to the receiver."><figcaption>Async by design — funds stay earning until the last moment. Relayers auto-claim once the request is ready.</figcaption></figure>
