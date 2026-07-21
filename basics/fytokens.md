# 🍯 fyTokens

**fyTokens are the ERC-4626 yield-bearing tokens you receive when you deposit into a ForgeYields vault.** One fyToken represents a proportional share of a vault's underlying assets — and of every Hallmark-underwritten strategy the vault is currently deployed across.

ForgeYields currently offers three vault tokens, each directional in its underlying: **fyUSDC** (USDC), **fyETH** (ETH), **fyWBTC** (WBTC). Each vault's composition — strategies, weights, current GRS per position — is published in real time and changes as the allocator rebalances. Every strategy in the mix must have a Hallmark score and clear the eligibility cutoff (GRS ≤ 7.5) before the allocator can deploy to it.

## What they do

* **Auto-compound.** Rewards accrue to the share price. The token count stays the same; the value of each token grows.
* **Liquid.** No lockup. Hold, transfer, or trade fyTokens anytime — they're standard ERC-4626 receipts.
* **Composable.** Use as collateral in lending markets, integrate into other protocols, bridge across L2s. Your yield position is a usable asset, not a frozen deposit.
* **Cross-chain.** Deposit from any supported chain; the vault settles on Ethereum. You receive fyTokens on the chain you deposited from.
* **Asynchronous redemption.** Withdraw via a request-then-claim flow. Funds stay earning until the next clearing cycle (typically minutes to hours, depending on liquidity). See [Redeem](../user-guide/redeem.md) for the user flow.

## What backs them

Every fyToken's value depends on the strategies the underlying vault is allocated to. Those strategies are:

* Pre-cleared by [Hallmark](../hallmark/overview.md) (GRS ≤ 7.5)
* Continuously monitored — a score that drifts above the cutoff triggers exit, not optimization
* Fully transparent — every score, every methodology version, every cascade dependency is [publicly verifiable](../hallmark/transparency.md)

You're not holding an IOU against an unknown set of farms. You're holding a share in a vault whose entire allocation universe is published and audited.

## Why this matters

Most yield aggregators give you a token that grows in value with no insight into what's behind it. fyTokens come with a published risk score for every strategy that contributes to the share price — and an immediate alert if that score degrades. This is the institutional bar applied to retail-accessible vaults.
