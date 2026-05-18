# Transparency & Public Scores

Hallmark is designed to be **independently verifiable**. Every underwriting decision has a published artifact, every methodology change has a dated amendment, and every score-to-allocation link is auditable. Frontier yield requires frontier transparency.

## What's public

| Artifact | Format | Purpose |
|---|---|---|
| **Methodology** | Versioned markdown | The rubric itself — every criterion, weight, and scoring band |
| **Amendments** | Dated markdown (`v3.5_amendment.md`, `v4.1_amendment.md`, …) | Methodology changes with rationale and backwards-compatibility notes |
| **Scores** | YAML, one file per protocol / asset / strategy / chain | The canonical machine-readable scores consumed by the allocator |
| **Assessments** | Markdown, one per protocol / asset / strategy | The human-readable analysis behind each score |

## Score format

Each scored entity has a YAML file in the public [`forge-hallmark`](https://github.com/ForgeYields/forge-hallmark) repository at `scores/{protocols|assets|strategies|chains}/{slug}.yaml`. A score record contains, at minimum:

```yaml
slug: morpho-blue
layer: protocol
methodology_version: "v4.1"
scored_at: "2026-05-18"
scored_by: "Risk Analyst Agent"
criteria:
  C1: { score: 2, evidence: "Spearbit + OpenZeppelin + ChainSecurity audits, all findings resolved" }
  C2: { score: 1, evidence: "TVL > $1B, 30d drawdown < 5%" }
  C3: { score: 3, evidence: "3/5 multisig + 48h timelock" }
  C4: { score: 2, evidence: "Zero exploits, > 18mo operational" }
  C5: { score: 3, evidence: "Singleton architecture, minimal dependencies" }
  C6: { score: 2, evidence: "Doxxed team, reputable track record" }
final_score: 2.3
```

Every score has **evidence**. A score without evidence is not a valid score.

## Validation

Hallmark ships a set of validation scripts that run on every score change:

- **`validate-scores.js`** — schema, range, and required-field checks on every YAML
- **`check-cascade-integrity.js`** — enforces the Recursive Strategy Collateral Rule: if you change an L1 score, every L3 strategy in its dependency tree must be re-scored
- **`check-drift.js`** — flags scores older than the cadence threshold (quarterly for L1/L2, event-driven for L3)

Allocator integration refuses scores that fail validation. There is no "manual override" path that bypasses these checks.

## Audit trail

For any historical allocation, the chain of evidence is:

```
Vault position
  ↓ which strategy?
Strategy YAML (GRS at deployment time)
  ↓ what methodology?
Methodology version (v4.1)
  ↓ what inputs?
Protocol score YAML + Asset score YAML + Strategy assessment markdown
```

Every link is timestamped and version-pinned. If a position later goes bad, post-mortem can attribute the failure to a specific criterion (e.g. "C4 was underweighted; team failed to disclose a prior incident") rather than to vague "model failure."

## How to verify a score yourself

1. Find the strategy in the [Strategies API](../integration/api/strategies-info.md).
2. Pull its current `score_id` and look up the corresponding YAML in the public Hallmark feed.
3. Cross-check the `criteria` block against the methodology version it references.
4. Read the assessment markdown for the qualitative reasoning.

If you find a discrepancy, raise it — Hallmark treats criticism as a feature, not a bug.

## What's not public (and why)

A subset of inputs to scoring is non-public: private security disclosures shared under NDA, team interviews, and operational due diligence notes. These influence the *evidence* fields but are never the sole basis for a score — every score must be reconstructible from public information alone. Private inputs can only tighten a score (push it higher / more conservative), never loosen it.
