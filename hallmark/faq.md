# ❓ FAQ

Quick answers to the questions institutional LPs and integrators ask most. For the full methodology see [Methodology](methodology.md); for the deployment rules see the [Allocator Policy](allocator-policy.md); for the public score feed see [Transparency](transparency.md).

***

## The basics

### What is Hallmark, in one sentence?

The open scoring protocol that measures the risk of every protocol, asset, chain, and strategy ForgeYields touches — published, versioned, and reproducible from public data.

### Who decides what the vaults hold — Hallmark or ForgeYields?

Both, in two separate public steps. **Hallmark measures**: it publishes scores and classification labels, and deliberately publishes no verdicts, thresholds, or caps. **ForgeYields decides**: its published [Allocator Policy](allocator-policy.md) consumes those scores and derives binary APPROVED / EXCLUDED verdicts, cap-bands that tighten allocation as scores rise, concentration ceilings, exit rules, and reassessment cadence.

The split means the same Hallmark scores would produce different deployment decisions at a more or less conservative allocator — that's by design. Any institution can adopt Hallmark and write its own policy; ForgeYields' Policy is the reference implementation, and it's as public as the scores it consumes.

### What's the eligibility cutoff?

Hallmark has none — a score is a measurement, not a verdict. Eligibility is defined by the [Allocator Policy](allocator-policy.md), which derives a binary APPROVED / EXCLUDED verdict from the live scores and labels: composite-score thresholds per layer, plus non-compensable hard triggers on individual criteria (a catastrophic governance or audit score excludes regardless of how good the rest looks). The exact threshold tables live in the Policy document and its machine-readable JSON — the same file the allocator consumes.

### Who uses Hallmark?

Three audiences:
1. **ForgeYields' allocator** — via the Allocator Policy, the binding gate on every deployment
2. **Integrators and LPs** — to verify what underwrites the yield they're holding
3. **The DeFi community** — anyone can pull the public score feed and apply their own policy to it

### Is Hallmark open-source?

The methodology, scores, schemas, validators — and ForgeYields' Allocator Policy — are all public:
- **Scores:** [github.com/ForgeYields/forge-hallmark](https://github.com/ForgeYields/forge-hallmark)
- **Methodology:** `/methodology/` (the current version is stamped at the top of `full-framework.md`, with the amendments timeline in `/methodology/amendments/`)
- **Allocator Policy:** `/policies/` — prose plus policy-as-code JSON
- **Validators:** `scripts/validate-scores.js`, `check-cascade-integrity.js`, `check-drift.js`

The raw assessment notes (qualitative analysis behind each score) are kept private under NDA where they reference confidential security disclosures. Public scores are reconstructible from public information alone — private inputs can only *tighten* a score, never loosen it.

***

## How scoring works

### Why is strategy-specific risk weighted highest in the GRS?

The GRS composes protocol, asset, and strategy-specific risk with published weights (current values in the [Methodology](methodology.md)):

- **Protocol risk** carries heavy weight because protocol failure is the most common loss vector empirically (exploit, governance compromise, oracle break) — but not dominant, because asset and execution risk can independently sink a position.
- **Asset risk** is real but typically slower-moving (depeg, backing degradation), and the Allocator Policy adds tighter caps and monthly monitoring on higher-risk asset bands.
- **Strategy-specific risk** carries the highest weight because it captures *what you're actually doing*. A great protocol holding a great asset in a stupid strategy is still a bad position.

The weights were not chosen via backtesting (which would overfit). They reflect operational priors from observing DeFi incidents and hundreds of strategy assessments — and they are stress-tested after the fact (see "How do you know the methodology works?" below).

### What happened to the WATCHLIST band?

It was retired when scoring and policy were split. There is no intermediate verdict anymore — everything is APPROVED or EXCLUDED under the Allocator Policy.

What remains is a *sizing rule*, not a verdict: approved positions whose scores fall in the upper bands (for example the 6.0–6.5 asset band) bind to a tighter allocation cap with monthly re-scoring and documented monitoring triggers. Cap-bands are described by the score band that produces them — current values in the [Allocator Policy](allocator-policy.md). A position in a tighter cap-band isn't "on watch"; it's approved, sized smaller, and looked at more often.

### What's "cascade integrity"?

A strategy's GRS depends on the protocols and assets it uses. If a protocol gets rescored (e.g., an exploit on Aave), every strategy that uses Aave needs to be re-evaluated. The `check-cascade-integrity.js` validator enforces this: an L1 rescore commit must be accompanied by L3 rescores for every dependent strategy. No orphaned scores allowed.

On the decision side, the Allocator Policy adds cascade *rules*: an exclusion anywhere in the dependency tree — protocol, asset, underlying, even a backing counterparty — flows to every strategy that touches it, regardless of how good the strategy's own raw score is.

***

## Cadence and changes

### How often are scores updated?

Reassessment cadence is a Policy setting, not a Hallmark constant — ForgeYields' current cadence table lives in the [Allocator Policy](allocator-policy.md). The shape of it:

- **Baseline cadence per layer** — periodic full re-evaluation, with higher-risk score bands and certain asset classes on a faster (monthly) cycle.
- **Material-event triggers** — depegs, missed attestations, exploits, custodian changes, and qualifying governance design-changes force an immediate re-score ahead of cadence.
- **Cascades** — any L0/L1/L2 re-score triggers re-evaluation of every dependent strategy.

The `check-drift.js` validator flags scores older than their cadence threshold.

### What happens if a score changes after I deposit?

If the position remains APPROVED under the Policy: the allocator may resize it on the next rebalance if the new score lands in a different cap-band, but the position stays.

If the Policy verdict flips to EXCLUDED — whether from the strategy's own score or a cascade from a dependency: the Policy's exit rules take over, with timelines scaled to the urgency of the trigger. Your fyToken share price reflects the unwound position.

Score changes are visible in real time in the public feed, and the corresponding exit appears in the [Atomic Transparency Ledger](../basics/atomic-transparency-ledger.md) with its policy rationale attached.

### Can I propose a score change?

Yes. Open an issue or PR on [forge-hallmark](https://github.com/ForgeYields/forge-hallmark) with evidence. Score changes require:
1. Documented evidence (audit report, on-chain incident, governance vote, etc.)
2. Methodology compliance (which criterion(s) are affected, why)
3. Cascade implications (which dependent L3 strategies need rescoring)

The Risk Analyst reviews and merges. Methodology changes (vs individual scores) require a new amendment; Allocator Policy changes version separately, each with its own changelog and archived predecessor.

***

## Trust and evidence

### How do I verify the rules myself?

Everything the allocator obeys is a public file — verify the rules the same way you verify a score:

1. **Read the Policy prose:** the [Allocator Policy](allocator-policy.md) page, and the canonical document in [`forge-hallmark/policies/`](https://github.com/ForgeYields/forge-hallmark/tree/main/policies) — verdict triggers, cap-band tables, exit timelines, cadence.
2. **Read the policy-as-code:** the same directory holds the machine-readable policy JSON — the *same file the allocator consumes*. There is no private rulebook behind the public one.
3. **Pull the live scores** from `scores/` and apply the Policy's threshold tables yourself: derive the verdict and cap-band for any strategy and compare against what the vaults actually hold.
4. **Check the ledger:** every movement in the [Atomic Transparency Ledger](../basics/atomic-transparency-ledger.md) carries its policy rationale — match it against the rule you just read.

If your derivation disagrees with a live position, that's a finding — open an issue.

### How do you know the methodology works?

It's tested against history and adversaries, not asserted:

- **28-incident no-hindsight backtest** — the scoring rules and Policy were replayed against major DeFi failures (Terra, Euler, Curve/Vyper, Maple, Radiant, and more) using only information available *before* each incident, to measure what would have been excluded, capped, or missed.
- **Adversarial reviews** — full sweeps of the live registries (every scored strategy, asset, and protocol) hunting for rules that misfire, and for live positions that a proposed rule would wrongly hit.
- **Zero-false-positive acceptance test** — the ratified amendment set caught 21 of 23 target incident classes with zero false positives on the live registry: no rule was accepted that would have wrongly excluded a sound position.
- **Annual performance study** — a standing commitment to publicly measure realized outcomes against what the scores predicted, every year.

Just as important is what the evidence process *refuses* to claim: the Policy's own LP disclosure states plainly that no scoring rule can catch every failure class, and that concentration caps — not scores — are the bound on unauditable-in-advance events. A methodology that claims otherwise is overfitted.

***

## Specific concepts

### What's a Type W wrapper vault?

A strategy that delegates execution to a third-party permissioned vault (Ipor Fusion, MetaMorpho, Yearn V3, generic ERC-4626 wrappers).

Type W exists because the trust model is fundamentally different from direct execution: you're not just trusting Aave's contracts, you're also trusting the curator/atomist who decides what the wrapper does. Hallmark scores this with the X1–X5 rubric:

| Criterion | What it measures |
|---|---|
| **X1** Underlying Strategy Risk | What the wrapper does economically |
| **X2** Curator/Atomist Trust | Who can move funds (EOA / MPC / multisig + timelock) |
| **X3** Exit Mechanism | Atomic / queue / pause history |
| **X4** Fee Structure | Transparent vs unilateral change |
| **X5** Vault Maturity | Track record, TVL, incidents |

X2 carries the heaviest weight (current values in the [Methodology](methodology.md)) because curator trust is the defining differentiator between a robust wrapper and a single-EOA-can-rug situation.

### What's CRS (Chain Risk Score)?

Layer 0 of the framework: every chain is scored independently using the N1–N5 rubric (consensus, sequencer, operational track record, bridge, VM maturity), and chain risk flows into the strategies deployed there through the protocol-risk composition.

This closes the gap where the same Morpho deployment on Ethereum vs Monad would otherwise score identically. See [Layer 0 — Chains](layer-0-chains.md).

### What's VRS (Vault Risk Score)?

Vault-level risk: the **allocation-weighted average of strategy GRS** across a vault's current positions, plus a **coverage** percentage (fraction of allocation with current Hallmark scores).

Where GRS is per-strategy, VRS is per-vault. fyUSDC, fyETH, and fyWBTC each have a current VRS visible in the app — and the Allocator Policy caps it: each vault's weighted risk must stay under a published VRS ceiling, which binds the vault as a whole on top of the per-position caps.

***

## Verification

### How do I verify a score myself?

1. Pick a strategy from the [Strategies API](../integration/api/strategies-info.md) or the fyToken page
2. Find its YAML in the public Hallmark feed at `forge-hallmark/scores/strategies/<slug>.yaml`
3. Cross-check the `criteria` block against the methodology version it references
4. Read the assessment markdown for qualitative reasoning
5. If you find a discrepancy, raise an issue

If you want a worked example: see [How a score is built — gteUSDc Morpho](example-score.md). To verify the *decision* layered on top of a score, see "How do I verify the rules myself?" above.

### Is Hallmark itself audited?

Hallmark is a methodology, not a smart contract — it's verified by its inputs and outputs being public, not by a code audit. The validators (`validate-scores.js`, `check-cascade-integrity.js`, `check-drift.js`) are open-source and run on every change. Anyone can fork the methodology and apply it independently to verify our scores. The evidence base above — backtest, adversarial reviews, acceptance test, annual study — is the methodology's equivalent of an audit trail.

The smart contracts that hold and move vault funds are audited separately — see [Audits](../other/audits.md).

***

## For partners

### Can I plug Hallmark scores into my own product?

Yes. The score feed is freely readable from the public repo, or pull through the ForgeYields API (a dedicated Hallmark API endpoint is on the roadmap). Attribution requested but not required.

You can also go further than the scores: adopt the full framework and write your own Allocator Policy against it, with thresholds and caps matching your risk appetite. ForgeYields' [Policy](allocator-policy.md) and its machine-readable JSON are a working template.

### How does this differ from "audited"?

An audit is a one-time review of code. Hallmark is a continuous, methodology-driven assessment of *operational* and *strategy* risk — most of which is invisible to a code audit.

A protocol can have 5 audits and still score poorly on protocol risk (e.g., because it relies on off-chain CEX hedging). Conversely, a protocol with fewer audits can score well if its operational profile is otherwise strong. The two are complementary, not substitutes.
