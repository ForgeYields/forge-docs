# Strategies Info

#### Endpoint

```
 GET https://api.forgeyields.com/strategies/strategies-info
```

***

#### Example response

```json
[
  {
    "name": "ForgeYields ETH",
    "symbol": "fyETH",
    "underlyingSymbol": "ETH",
    "decimals": 18,
    "projectedApy": "8.05",
    "realisedApy": "6.72",
    "averageRedeemDelay": 43083.92, 
    "redeemFees": "0.0005", 
    "tvl": "46.915517488557815777",
    "tvlUsd": "141184.98497921418",
    "sharePrice": "1.019937",
    "tokenGatewayPerDomain": [
      {
        "domain": "starknet",
        "tokenGateway": "0x050707bc3b8730022f10530c2c6f6b9467644129c50c2868ad0036c5e4e9e616"
      }
    ]
  }
]
```

#### What this includes

* Strategy name & symbol
* Underlying token (`ETH`, `USDC`, …)
* Decimals (underlying and token\_gateay tokens always share same decimals)
* Projected APY (based on current allocation)
* Realised APY (7d average share price evolution)
* Average exit (redeem) delay in seconds, based on historical data.
* Redeem fees
* TVL (underlying + USD)
* Share price
* Token Gateways per domain
