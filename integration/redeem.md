# Redeem

### 1. Redeem flow (generic)

Redemptions in ForgeYields are **asynchronous** and follow a **request → claim** model.

1. The user calls **`request_redeem(shares, owner, receiver)`** on the Token Gateway.
2. The fyTokens are **burned**, and an ERC-721 redeem request is **minted**, recording:
   * the number of shares,
   * the epoch at which the request was created,
3. During the next clearing cycle, the withdrawal pool is processed and the request becomes **claimable**.
4. Anyone can then call **`claim(requestId, receiver)`**.\
   This burns the redeem NFT and transfers the corresponding underlying assets to the receiver.

#### Generic flow

```
TOKEN_GATEWAY.requestRedeem(shares, user, user)
→ redeem request created (ERC-721)

<withdrawal pool get processed>

TOKEN_GATEWAY.claim(requestId, user)
→ user receives underlying
```

#### Notes

* No approve required, `requestRedeem` does not need a prior approval, fyTokens are burned internally.
* Underlying is delivered on claim, the actual transfer of assets happens only during the `claim` step.
* Claim is handled by ForgeYields relayers, we consume your request as soon as it is ready.
* You can fetch pending redeem requests using the [API](api/pending-redeems.md).
* Redeem fees might exists, you can fetch it via [API](api/strategies-info.md)**.**

***

### 2. Starknet example (starknet.js)

```ts
import { Provider, Account, Contract } from "starknet";

const provider = new Provider({ rpc: { nodeUrl: STARKNET_RPC_URL } });
const account = new Account(provider, USER_ADDRESS, USER_PRIVATE_KEY);

// Addresses from the strategy page (Deployment → <chain> Token Gateway)
const TOKEN_GATEWAY_ADDRESS = "0xGATEWAY...";

// Minimal ABI for the Token Gateway
const tokenGatewayAbi = [
  "function request_redeem(uint256 shares, address owner, address receiver)"
];

// Contract instance
const tokenGateway = new Contract(tokenGatewayAbi, TOKEN_GATEWAY_ADDRESS, account);

// Example: redeem 0.5 fyToken (decimals is always the same than fyToken)
const shares = 500000000000000000n; // 0.5 fyToken

async function requestRedeem() {
  const calls = [
    await tokenGateway.populate("request_redeem", [
      shares,             // fyToken amount (shares)
      account.address,    // owner
      account.address     // receiver of underlying
    ]),
  ];

  const tx = await account.execute(calls);
  console.log("Multicall redeem request tx hash:", tx.transaction_hash);
}
```

***

### 3. EVM example (ethers.js)

```ts
// EVM example (ethers v6)
import { ethers } from "ethers";

const provider = new ethers.JsonRpcProvider(EVM_RPC_URL);
const signer = new ethers.Wallet(USER_PRIVATE_KEY, provider);

// Addresses
const TOKEN_GATEWAY_ADDRESS = "0xGATEWAY...";

const tokenGatewayAbi = [
  "function requestRedeem(uint256 shares, address owner, address receiver) external"
];

const tokenGateway = new ethers.Contract(TOKEN_GATEWAY_ADDRESS, tokenGatewayAbi, signer);

async function requestRedeem() {
  const shares = ethers.parseUnits("0.5", 18); // redeem 0.5 fyToken

  const user = await signer.getAddress();

  // Direct call — no approve required
  const tx = await tokenGateway.requestRedeem(shares, user, user);
  await tx.wait();

  console.log("Redeem request submitted");
}
```

***

### 4. Optional: use a swap aggregator for **instant withdrawal**

Instead of going through `request_redeem`, you can exit immediately by **swapping fyTokens for the underlying** on the same chain.

How it works:

* Query a swap aggregator for a `fyToken → underlying` route.
* Aggregators offer **instant settlement**, but usually at a **worse price** than the asynchronous redeem pipeline.
* If the user prefers speed over optimal price, you can bypass the redeem process entirely.

Two withdrawal paths:

**Base path (best execution, slower):**\
`request_redeem(shares, user, user)` → wait for withdrawal pool to be processed → `claim(requestId, user)`

**Instant path (faster but might involve a loss):**\
Swap `fyToken → underlying` through a DEX/aggregator for immediate exit.
