# ⚠️ Risks

**ForgeYields smart-contract risk**\
Even audited contracts may contain bugs. That said, the attack surface is intentionally minimal: contracts only expose deposit and redemption requests, and redemptions are asynchronous. This design makes direct liquidity extraction from a single chain impossible, limiting the impact of potential exploits.

**Protocol risk**\
Vulnerabilities in underlying protocols used by the strategies (Curve, Convex, LST/LRT issuers, lending markets, etc.).

**Liquidity risk**\
During high demand or volatile periods, redemptions may take longer until liquidity is freed during the next rebalance.

**Bridge risk**\
Even though ForgeYields defaults to canonical bridges, all cross-chain operations carry technical and operational execution risk.

