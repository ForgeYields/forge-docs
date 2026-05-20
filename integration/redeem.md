# 💸 Redeem

### 1. Redeem flow (generic)

Redemptions in ForgeYields are **asynchronous** and follow a **request → claim** model.

1. The user calls **`requestRedeem(shares, receiver, owner)`** on the Token Gateway. The fyTokens are **burned**, and an ERC-721 redeem request is **minted**, recording the number of shares and the epoch at which the request was created. The function returns a redemption `id`.
2. During the next clearing cycle, the withdrawal pool is processed and the request becomes **claimable**.
3. Anyone can then call **`claimRedeem(id)`** (Cairo: `claim_redeem(id)`). This burns the redeem NFT and transfers the underlying assets to the receiver that was set when the request was created.

#### Generic flow

```
TOKEN_GATEWAY.requestRedeem(shares, receiver, owner) → returns id
→ redeem request created (ERC-721)

<withdrawal pool gets processed>

TOKEN_GATEWAY.claimRedeem(id) → returns assets
→ receiver gets the underlying
```

#### Notes

* **No approve required** — `requestRedeem` does not need a prior approval; fyTokens are burned internally.
* **Receiver is locked at request time** — the address that will receive the underlying is set when the request is created, not when the claim happens.
* **Claim has only one parameter** — the `id` returned by `requestRedeem`. There is no separate `receiver` argument on the claim call.
* **Auto-claim by relayers** — ForgeYields relayers consume your request as soon as it's ready; you typically don't need to call `claimRedeem` yourself.
* Fetch pending redeem requests using the [API](api/pending-redeems.md).
* Redeem fees may apply — fetch via the [strategies-info API](api/strategies-info.md).

***

### 2. Starknet example (starknet.js)

```ts
import { Provider, Account, Contract } from "starknet";

const provider = new Provider({ rpc: { nodeUrl: STARKNET_RPC_URL } });
const account = new Account(provider, USER_ADDRESS, USER_PRIVATE_KEY);

// Address from the API: see api/strategies-info.md → tokenGatewayPerDomain
const TOKEN_GATEWAY_ADDRESS = "0xGATEWAY...";

// Minimal ABI for the Token Gateway
const tokenGatewayAbi = [
  "function request_redeem(uint256 shares, address receiver, address owner) -> uint256",
  "function claim_redeem(uint256 id) -> uint256"
];

const tokenGateway = new Contract(tokenGatewayAbi, TOKEN_GATEWAY_ADDRESS, account);

// Example: redeem 0.5 fyToken
const shares = 500000000000000000n; // 0.5 fyToken (18 decimals)

async function requestRedeem() {
  const tx = await account.execute([
    tokenGateway.populate("request_redeem", [
      shares,             // fyToken amount (shares)
      account.address,    // receiver of underlying (locked at request time)
      account.address     // owner of the fyTokens being burned
    ]),
  ]);
  console.log("Redeem request tx hash:", tx.transaction_hash);
}

// Manual claim (usually unnecessary — relayers handle this automatically)
async function claimRedeem(redeemId: bigint) {
  const tx = await account.execute([
    tokenGateway.populate("claim_redeem", [redeemId]),
  ]);
  console.log("Claim tx hash:", tx.transaction_hash);
}
```

***

### 3. EVM example (ethers.js)

```ts
import { ethers } from "ethers";

const provider = new ethers.JsonRpcProvider(EVM_RPC_URL);
const signer = new ethers.Wallet(USER_PRIVATE_KEY, provider);

const TOKEN_GATEWAY_ADDRESS = "0xGATEWAY...";

const tokenGatewayAbi = [
  "function requestRedeem(uint256 shares, address receiver, address owner) external returns (uint256 id)",
  "function claimRedeem(uint256 id) external returns (uint256 assets)"
];

const tokenGateway = new ethers.Contract(TOKEN_GATEWAY_ADDRESS, tokenGatewayAbi, signer);

async function requestRedeem() {
  const shares = ethers.parseUnits("0.5", 18); // redeem 0.5 fyToken
  const user = await signer.getAddress();

  // Direct call — no approve required
  const tx = await tokenGateway.requestRedeem(
    shares,
    user,  // receiver of underlying
    user   // owner of fyTokens being burned
  );
  const receipt = await tx.wait();
  console.log("Redeem request submitted");
}

// Manual claim (usually unnecessary — relayers handle this automatically)
async function claimRedeem(redeemId: bigint) {
  const tx = await tokenGateway.claimRedeem(redeemId);
  await tx.wait();
  console.log("Claimed");
}
```

***

### 4. Optional: use a swap aggregator for **instant withdrawal**

Instead of going through `requestRedeem`, you can exit immediately by **swapping fyTokens for the underlying** on the same chain.

How it works:

* Query a swap aggregator for a `fyToken → underlying` route.
* Aggregators offer **instant settlement**, but usually at a **worse price** than the asynchronous redeem pipeline.
* If the user prefers speed over optimal price, you can bypass the redeem process entirely.

Two withdrawal paths:

**Base path (best execution, slower):**\
`requestRedeem(shares, receiver, owner)` → wait for withdrawal pool to be processed → `claimRedeem(id)` (typically auto-handled by relayers)

**Instant path (faster but might involve a loss):**\
Swap `fyToken → underlying` through a DEX/aggregator for immediate exit.
