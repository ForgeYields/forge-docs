# Pending redeems

#### **Endpoint**

```http
 GET https://api.forgeyields.com/strategies/pending-redeems/:domain/:address
```

**Path params**

* `domain` – domain identifier (e.g. `starknet`, `ethereum`)
* `address` – user wallet address on that domain

***

#### **Example response**

```json
[
  {
    "epoch": 74,
    "shares": "162.989203",
    "assets": "166.523679",
    "redeemId": "15",
    "timestamp": 1764277625,
    "transactionHash": "0x03f5...3335",
    "strategy": "fyUSDC"
  }
]
```

***

#### Field description

* **epoch**\
  The global epoch during which the redeem request was recorded.
* **shares**\
  Amount of fyTokens the user requested to redeem.
* **assets**\
  Underlying assets the user will receive.
* **redeemId**\
  Internal redeem identifier.
* **timestamp**\
  Unix timestamp (in seconds) when the request was submitted.
* **transactionHash**\
  Hash of the on-chain transaction that created the redeem request.
* **strategy**\
  The strategy symbol (`fyETH`, `fyUSDC`, `fyWBTC`, …).
