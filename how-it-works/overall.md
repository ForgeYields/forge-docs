---
description: Understand ForgeYields cross-chain architecture
---

# 🌐 Overall

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

### Architecture Breakdown

#### **1. User Interface Layer (Token Gateways on Each Chain)**

Users interact with ForgeYields through Token Gateways deployed on each supported chain (Starknet, Ethereum, Scroll). These gateways:

* Receive **deposit** and **redeem** requests
* Perform **local netting**
* Handle cross-chain transfers through canonical bridges
* Forward asset flows and reports to the global Controller

This layer abstracts all multi-chain complexity. Users only interact on their chain of choice while the system handles settlement across others.

***

#### **2. Coordination Layer (Controller on Starknet)**

The Controller contract on Starknet is the global coordination hub. It:

* Receives **Token Gateway Reports** (deposit totals, redemption totals)
* Receives **Vault Reports** (executed trades, AUM updates)
* Calculates global AUM and rollup-specific deltas
* Sends back the **Controller Report** to all Token Gateways
* Triggers settlement across chains

This layer also uses an **Oracle Role** that submits off-chain prices, AUM snapshots, and cross-chain state needed to maintain the global epoch logic.

The Controller is intentionally isolated and minimalist, improving auditability and reducing attack surface.

***

#### **3. Strategy Execution Layer (Manager + Vault on Ethereum)**

This layer is where capital is allocated and yield is generated.

**Vault (Ethereum):**

* Receives assets bridged from Token Gateways
* Deposits into underlying Ethereum DeFi protocols (Curve, LST/LRT issuers, lending markets)
* Executes trades and rebalances liquidity
* Sends Vault Reports back to the Controller

**Manager (Ethereum):**

* Enforces the **Merkle-based allowed calls** (set off-chain by the Strategist)
* Ensures only vetted integrations and methods can be executed
* Acts as the “instruction layer” for strategy changes

**Strategist (Off-chain engine):**

* Computes the optimal allocation
* Allowed actions via Merkle roots predefined&#x20;
* Sends instructions through the Manager to perform swaps, deposits, and reallocations

This layer captures incentives, rotates positions, and runs the automated yield optimization logic of ForgeYields.

***

#### **4. Mailboxes (Hyperlane) Across Chains**

At the top of each layer, Mailboxes (Starknet, Ethereum, Scroll) handle:

* Cross-chain messaging
* Report synchronization
* Controlled dispatch between Token Gateways, Controller, and Vault

This enables a **fully distributed yet unified global state** across all chains.

***

### Summary

This architecture separates ForgeYields into three clean layers:

* **User Interface Layer** → local deposit/redeem, chain abstraction
* **Coordination Layer** → global AUM, epochs, cross-chain state
* **Strategy Execution Layer** → yield farming, incentive chasing, allowed calls

Each component has a minimal, clearly defined responsibility, ensuring security, scalability, and fast strategy upgrades.
