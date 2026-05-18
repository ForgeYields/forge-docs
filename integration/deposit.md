# Deposit

### 1. Deposit flow (generic)

The Token Gateway (contract handling deposits and redeems) implements the ERC-4626 standard:

1. Approve the **underlying token** to the Token Gateway.
2. Call:

* `deposit(assets, receiver)`: deposit a fixed amount of underlying
* `deposit_referral(assets, receiver, referral_code)` : deposit variant with referral
* `mint(shares, receiver)`: mint a fixed amount of fyTokens.
* `mint_referral(shares, receiver, referral_code)`: mint variant with referral.

#### Generic flow

```
underlying.approve(token_gateway, amount)
token_gateway.deposit_referral(amount, user, referral_code)
→ User receives fyTokens
```

* You can get the `token_gateway` address via our [API](api/strategies-info.md).
* The protocol does not charge any deposit fees.&#x20;
* Communicate your `referral_code` with **the** ForgeYields team. Revenue from depositors is credited to your integration.&#x20;

### 2. Starknet example (starknet.js)

```ts
import { Provider, Account, Contract } from "starknet";

const provider = new Provider({ rpc: { nodeUrl: STARKNET_RPC_URL } });
const account = new Account(provider, USER_ADDRESS, USER_PRIVATE_KEY);

const UNDERLYING_ADDRESS = "0xUNDERLYING...";
const TOKEN_GATEWAY_ADDRESS = "0xGATEWAY...";

const REFERRAL_CODE = 123456n; // felt252 given by ForgeYields

const erc20Abi = [
  "function approve(address spender, uint256 amount)"
];

const tokenGatewayAbi = [
  "function deposit(uint256 assets, address receiver) returns (uint256 shares)",
  "function mint(uint256 shares, address receiver) returns (uint256 assets)",
  "function previewMint(uint256 shares) view returns (uint256 assets)",
  "function deposit_referral(uint256 assets, address receiver, felt252 referral_code) returns (uint256 shares)",
  "function mint_referral(uint256 shares, address receiver, felt252 referral_code) returns (uint256 assets)"
];

const underlying = new Contract(erc20Abi, UNDERLYING_ADDRESS, account);
const tokenGateway = new Contract(tokenGatewayAbi, TOKEN_GATEWAY_ADDRESS, account);

const amount = 1_000000000000000000n; // 1 underlying token

async function depositFyTokens() {
  await account.execute([
    underlying.populate("approve", [TOKEN_GATEWAY_ADDRESS, amount]),
    tokenGateway.populate("deposit_referral", [
      amount,
      account.address,
      REFERRAL_CODE
    ])
  ]);
}
```

#### Mint variant

```ts
async function mintFyTokens(desiredShares: bigint) {
  const requiredAssets = await tokenGateway.call("previewMint", [desiredShares]);
  const required = BigInt(requiredAssets);

  await account.execute([
    underlying.populate("approve", [TOKEN_GATEWAY_ADDRESS, required]),
    tokenGateway.populate("mint_referral", [
      desiredShares,
      account.address,
      REFERRAL_CODE
    ])
  ]);
}
```

***

### 3. EVM example (ethers.js)

```ts
import { ethers } from "ethers";

const provider = new ethers.JsonRpcProvider(EVM_RPC_URL);
const signer = new ethers.Wallet(USER_PRIVATE_KEY, provider);

const UNDERLYING_ADDRESS = "0x...";
const TOKEN_GATEWAY_ADDRESS = "0x...";

const REFERRAL_CODE = 123456n; // uint256 provided by ForgeYields

const erc20Abi = [
  "function approve(address spender, uint256 amount) external returns (bool)"
];

const tokenGatewayAbi = [
  "function deposit(uint256 assets, address receiver) external returns (uint256 shares)",
  "function mint(uint256 shares, address receiver) external returns (uint256 assets)",
  "function previewMint(uint256 shares) external view returns (uint256 assets)",
  "function depositReferral(uint256 assets, address receiver, uint256 referralCode) external returns (uint256 shares)",
  "function mintReferral(uint256 shares, address receiver, uint256 referralCode) external returns (uint256 assets)"
];

const underlying = new ethers.Contract(UNDERLYING_ADDRESS, erc20Abi, signer);
const tokenGateway = new ethers.Contract(TOKEN_GATEWAY_ADDRESS, tokenGatewayAbi, signer);

const amount = ethers.parseUnits("1.0", 18);

async function depositFyTokens() {
  await underlying.approve(TOKEN_GATEWAY_ADDRESS, amount);

  await tokenGateway.depositReferral(
    amount,
    await signer.getAddress(),
    REFERRAL_CODE
  );
}
```

#### Mint variant

```ts
async function mintFyTokens() {
  const desiredFyShares = ethers.parseUnits("1.0", 18);

  const requiredAssets = await tokenGateway.previewMint(desiredFyShares);

  await underlying.approve(TOKEN_GATEWAY_ADDRESS, requiredAssets);

  await tokenGateway.mintReferral(
    desiredFyShares,
    await signer.getAddress(),
    REFERRAL_CODE
  );
}
```

***

### 4. Optional: aggregator-based entry

You can also enter a strategy by swapping directly into fyTokens via an on-chain DEX aggregator.\
Some markets trade fyTokens at a premium/discount.\
In those cases:

* compare aggregator output vs ERC-4626 deposit/mint (preview\_deposit/preview\_mint)&#x20;
* choose whichever gives the user more fyTokens
