# Audits

#### **1. Clearing Engine (ForgeYields) —** [**Audited**](https://github.com/ForgeYields/audits/blob/main/Forge%20-%20Csc%20Audit%20Report.pdf)

Handles:

* Deposits
* Withdrawals
* Batching & netting
* Cross-chain settlement

This component has an intentionally minimal attack surface:\
only deposit and redemption request functions are exposed, and redemption is asynchronous significantly reducing exploitability risks.

***

#### **2. Yield Allocator (EVM) —** [**Audited**](https://docs.veda.tech/audits)

ForgeYields uses the **Veda Labs Boring Vault** allocator for execution on EVMs.

The allocator enforces Merkle-validated allowed calls, ensuring the relayer can only interact with approved protocols, methods, and parameters.<br>
