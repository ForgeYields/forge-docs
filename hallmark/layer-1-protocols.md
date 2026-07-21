# Layer 1 — Protocols

Hallmark scores every protocol a strategy touches — Aave, Morpho, Curve, Pendle, and every other venue capital can sit in — as its own layer: **Layer 1**, the **Protocol Risk Score (PRS, 1–10)**. A protocol is scored once per deployment context, before any strategy built on it can be assessed; per-chain re-deployments inherit the underlying [chain risk](layer-0-chains.md) rather than being treated as new protocols.

> **Canonical rubric.** Criterion weights, scoring bands, and formulas live in the versioned [Layer 1 methodology file](https://github.com/ForgeYields/forge-hallmark/blob/main/methodology/layer1_protocol_assessment_methodology.md) — this page describes the structure only. If the two ever diverge, the canonical file wins.

## The PRS rubric (C1–C6)

Each protocol is scored on six C-criteria:

| Criterion | What it measures |
|---|---|
| **C1** Audit status | Audit count and tier, findings resolution, unaudited code delta; immutable-core protocols are not penalized for audit age |
| **C2** TVL history | Absolute TVL, drawdown, distance from highs |
| **C3** Governance quality | Multisig configuration, timelocks, upgrade controls, custody mode |
| **C4** Incident history & operational age | Past exploits, loss magnitude, remediation quality, track record |
| **C5** Smart contract risk | Complexity, dependencies, cross-chain configuration, off-chain reliance, known exploit lineages |
| **C6** Team & transparency | Doxxing status, track record, communication consistency |

Alongside the PRS, Layer 1 assessments feed the protocol-level [classification labels](methodology.md#classification-labels) — such as `custody_tier` (who can move funds) and `upgradeability` (is the deployed core immutable or upgradeable) — which consumer policies key their own rules to.

## How PRS connects to strategy GRS

The PRS is the protocol-risk component of every strategy's [GRS](methodology.md#the-four-layers). Chain risk enters through the same slot via the Multi-Protocol Rule, and multi-protocol strategies are penalized for compositional complexity — the exact composition formula is stated in the [Layer 3 methodology file](https://github.com/ForgeYields/forge-hallmark/blob/main/methodology/layer3_strategy_assessment_methodology.md). Any PRS change propagates to every strategy that depends on the protocol.

## Re-scoring cadence

Protocols are re-evaluated on the cadence set by the [Allocator Policy](allocator-policy.md) (§6):

- **Quarterly** baseline (full C1–C6 re-score); monthly once a protocol enters its higher score band
- **Event-driven** on material incidents — exploits at the protocol or its relatives, mechanism-design governance changes, custody changes, public security warnings

What a PRS *means* for deployment — verdicts, caps, exclusions — is decided by the [Allocator Policy](allocator-policy.md), not here. See [Transparency & public scores](transparency.md) for how to verify any PRS yourself from the public feed at `forge-hallmark/scores/protocols/*.yaml`.
