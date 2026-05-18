---
description: Understand ForgeYields, step by step
---

# 🔄 Step by Steps

#### 1. Deposit from Any Chain

Start by depositing your crypto into a ForgeYields strategy from any supported rollup. Your assets are pooled in a local buffer with other deposits.&#x20;

In return, you instantly mint a yield-bearing token -> fyToken

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

#### 2. Netting and Settlement

Deposits and withdrawals are **batched and netted** at the rollup level. Then, all local deltas are globally netted across chains and settled on Ethereum L1. This approach drastically reduces gas usage and avoids unnecessary bridge hops.<br>

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

#### 3. Ethereum Vault Settlement

Only the final net result is processed through Ethereum's canonical bridge. Assets are either deosited into or withdrawn from the main vault, ensuring all yield activity remains Ethereum-secured.

<figure><img src="../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

#### 4. Strategy Execution

ForgeYields allocation engine manages funds across different chains and protocols. All actions are strictly validated via a whitelist of allowed calls, and strategies are designed to maximize yield across top-tier DeFi projects.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

#### 5. Earn Yield Passively

Your **fyToken** grows in value automatically based on the performance of the underlying strategy. Meanwhile, you can freely use the token in DeFi, move it across chains, or explore new ecosystems all without touching the underlying assets or relying on third-party bridges.

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

#### 6. Redeem Anytime, from Anywhere

When you're ready, redeem your fyToken on any supported chain. The underlying assets are released once the vault settles liquidity, typically after the next strategy rebalancing.

> 🔁 **Redemptions are asynchronous** — your capital continues earning until the last moment, and the delayed settlement helps reduce exploit risk by limiting direct liquidity exposure.

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>
