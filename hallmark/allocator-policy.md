# Allocator policy

**Hallmark measures risk. The Allocator Policy decides what ForgeYields does about it.**

Hallmark is a pure scoring protocol: it publishes numerical risk scores and classification labels for chains, protocols, assets, and strategies — and deliberately publishes no verdicts, no caps, no exit rules. Those decisions belong to the **ForgeYields Allocator Policy**: a public, versioned document that translates Hallmark scores into deployment decisions according to ForgeYields' specific risk appetite. It is the reference implementation of a Hallmark consumer policy — any institution adopting Hallmark can write its own.

> **Where the numbers live.** Parameter values quoted on this page reflect the current published policy version. The canonical source is always the [policy document and its machine-readable mirror](https://github.com/ForgeYields/forge-hallmark/tree/main/policies) — on any divergence, those files win.

## Verdicts are binary

The policy emits exactly two verdict labels per protocol, asset, and strategy: **APPROVED** or **EXCLUDED**. There is no intermediate verdict. Elevated-but-acceptable risk is handled by allocation caps, not by a third label: a strategy in a higher score band is simply APPROVED with a tighter cap and a faster re-scoring cadence.

A verdict can be produced two ways:

- **Composite thresholds** — the layer's composite score (PRS, ARS, or GRS) crosses the policy's ceiling for that layer.
- **Hard triggers** — a single criterion crosses a non-compensable line, regardless of how good the composite looks. Examples: no credible audit on deployed code, a catastrophic incident history, anonymous governance holding drain-vector keys, a demonstrated sustained depeg, or majority-endogenous backing. The lesson behind hard triggers is that weighted averages dilute death sentences — an asset can composite to a respectable score while carrying one criterion that has historically been terminal.
- **Conditional triggers** — combinations of criteria that are individually survivable but jointly disqualifying (for example, a prior material incident combined with weak custody over upgradeable code). Linear composites cannot express conditional risk; these triggers can.

The exact threshold values and trigger tables live in the [policy document](https://github.com/ForgeYields/forge-hallmark/tree/main/policies).

**Verdicts are re-derived live.** Any `verdict` field in a published score file is informational only. The allocator re-derives every verdict at allocation time from the current scores and the live state of dependencies. A strategy with an excellent raw score whose dependency graph contains an EXCLUDED protocol is non-deployable — and becomes deployable again in the same cycle if the cascade lifts.

## Cascades: risk flows through dependencies

Verdicts follow the **weakest-link rule** through the dependency graph:

- **Layer cascades.** A strategy is EXCLUDED if any chain, protocol, or asset it depends on is EXCLUDED — no matter what its own GRS says.
- **Wrapper cascade.** A wrapped or vault-share asset is evaluated along both of its trust paths: the protocol that issues the wrapper *and* the underlying asset inside it. Either path EXCLUDED excludes the wrapper. Without this rule, an excluded underlying could be laundered through a wrapper whose issuer looks fine. The cascade evaluates recursively through nested wrappers.
- **Backing-counterparty cascade.** An asset is also evaluated against *where its backing actually sits*. If more than a threshold share of an asset's look-through backing is lent to (or held as trading margin at) a counterparty whose protocol verdict is EXCLUDED, the asset is EXCLUDED — even if that counterparty appears nowhere in the conventional dependency graph. This rule exists because of a real incident topology: a stablecoin had roughly two-thirds of its backing lent to a counterparty that was already excluded five times over on its own scores, yet was invisible to every dependency-based rule because it was neither the issuer nor the underlying. Credit exposure is a dependency; the policy now treats it as one.

When an active position flips to EXCLUDED, exit timelines apply — scaled by exposure size and severity, with the fastest lanes reserved for incident-grade triggers. For positions that are structurally locked (closed-end maturities, fully-utilized lending pools) after a *methodology-driven* re-score — as opposed to a real-world incident — a documented remediation framework governs the wind-down instead, with a public transparency note for large, slow exits.

## Concentration caps: the airbag

Verdicts answer "may capital go here at all?". Caps answer "how much?" — and they are the part of the policy that works even when scoring is wrong. In backtesting, every missed incident that stayed loss-bounded was bounded by a score-independent cap. Several cap families apply concurrently; **the binding cap is always the minimum**:

- **Score-band caps.** Each protocol, asset, and strategy carries a maximum vault allocation derived from its score band — tighter bands bind as scores worsen, down to zero at the EXCLUDED threshold. Strategy-type-specific sizing rules (pool-share limits, borrow-share limits, per-maturity limits) apply on top.
- **Absolute venue ceiling** *(currently 40% of a vault)*. No score maps to unlimited concentration — the best-scoring protocol imaginable in mid-2023 lost ~$70M a month later. Even the bluest chip is capped. Precedent: sovereign ceilings in credit ratings — even AAA sits under a systemic cap.
- **Shared-dependency cluster cap** *(currently 30% of a vault)*. Positions that share a hard dependency key — the same implementation family (factory lineage or shared bytecode) or the same non-dominant compiler toolchain — form a *cluster*, and the cluster is capped as if it were one position. This is the "two protocols sharing one compiler are one risk" rule: in 2023, four seemingly separate Curve pools failed together because they were compiled with the same vulnerable Vyper versions. Four positions, one dependency, one loss. Diversification that shares infrastructure is not diversification.
- **Regime-dependent-yield aggregate cap** *(currently 15% of a vault)*. Assets whose peg or principal depends on the continued profitability of a market-neutral trade (perp funding, basis capture) are capped *in aggregate across every strategy type* — lending collateral, LP legs, and PT underlyings all count toward one bucket, so the same exposure cannot be re-assembled across strategy types.
- **Vault risk ceiling.** The allocation-weighted average GRS of each vault must stay below a fixed ceiling *(currently 5.5)* — a portfolio-level bound on aggregate risk appetite, independent of any single position.
- **Label-driven tightenings.** Certain published Hallmark labels bind a position one cap-band tighter than its score alone would allow — for example, assets whose redemption value can be changed unilaterally by issuer governance. Multiple 15%-class bindings on the same asset resolve to the minimum once; tightenings do not stack multiplicatively.

## Re-scoring cadence and event triggers

Scores are only as good as their freshness. The policy sets the reassessment schedule (current baseline values — the canonical table is the policy's cadence section):

| Subject | Baseline cadence |
|---|---|
| Protocols | Quarterly; monthly once a protocol enters its higher score band |
| Assets | Monthly; a narrow blue-chip carve-out (canonical gas wrappers and attested custodial BTC/ETH wrappers) runs quarterly while attestations stay current, reverting to monthly instantly on any staleness |
| RWA assets | Cadence by class — T-bill class quarterly-acceptable, corporate-credit and insurance classes monthly with event monitoring |
| Regime-dependent-yield assets | Monthly, regardless of class or score band |
| Strategies | On material change (governance events, audit findings, sharp TVL moves, any dependency re-score) |
| Chains | Quarterly |

On top of the calendar, **material events trigger immediate re-scoring**: sustained peg deviations, missed or stale attestations, custodian changes, sharp liquidity drops, incidents at related protocols, changes in an asset's backing-counterparty composition, and public security warnings from recognized firms. Governance events are triggered with deliberate scope — a *mechanism-design change* (new mint/redemption contracts, a changed redemption formula, a parameter pushed outside its documented range) fires an immediate re-score; a routine parameter move within pre-authorized ranges does not. The boundary is the machine versus the dials: an unscoped "any governance vote" trigger would fire weekly and train everyone to ignore it.

## Policy-as-code

The policy is not just prose. It ships in three public files:

| File | Role |
|---|---|
| [`forgeyields_allocator_policy_v1_2.md`](https://github.com/ForgeYields/forge-hallmark/blob/main/policies/forgeyields_allocator_policy_v1_2.md) | The authoritative prose — every rule, with its evidence |
| [`allocator_policy_v1_2.json`](https://raw.githubusercontent.com/ForgeYields/forge-hallmark/main/policies/allocator_policy_v1_2.json) | Machine-evaluable mirror of the rules |
| [`allocator_policy.schema.json`](https://github.com/ForgeYields/forge-hallmark/blob/main/policies/allocator_policy.schema.json) | JSON Schema every policy JSON must validate against |

**The JSON the allocator runs is the same file you can read.** ForgeYields' backend fetches the policy JSON from the public mirror exactly as it fetches score files, validates it against the schema, and fails closed — if validation fails or a score predates the policy's minimum methodology version, no new allocation happens against it. Every numeric threshold in the JSON carries a `src` reference to the prose section it mirrors; on any divergence the prose wins and the JSON is a bug. Prose and JSON version-bump together — a policy amendment is one change: prose edit, JSON edit, changelog row.

Two honesty notes, encoded in the files themselves. First, an *activation* block records which rules are enforced and which are monitor-only pending data backfills — a trigger that reads an unpopulated field is a dead rule, and the policy says so rather than pretending otherwise. Second, the JSON is deliberately incomplete: rules that require human judgment (remediation adjudication, governance-event classification, counterparty affiliation) are flagged as prose-only, and positions whose eligibility turns on them are routed to human review, not auto-approved.

## Same scores, different allocators — by design

Hallmark scores are measurements; the policy is one institution's risk appetite applied to them. From the policy's own LP-communication section:

> "Hallmark scores assets according to a public risk methodology. ForgeYields' Allocator Policy then translates those scores into binary deployment decisions per our risk appetite (APPROVED or EXCLUDED), with cap-band bindings that tighten allocation as risk increases, absolute and cluster ceilings that bound concentration independently of scores, and cascade rules that follow risk through wrappers and backing counterparties. Both are public. The same Hallmark scores would produce different deployment decisions at a more or less conservative allocator — that's by design."

And the honest residual: a fully compliant vault can still lose meaningful value — on the order of 20–30% of TVL — in a single control-class event, the kind of unauditable-in-advance failure that has hit even the most audited blue-chip protocols. The concentration rules bound that class; no scoring rule can predict it, and a methodology that claims otherwise is overfitted. This is the irreducible price of being in the market, and the policy states it rather than hiding it.

## The evidence base

The current policy version was not designed in a vacuum:

- **A 28-incident, no-hindsight backtest.** Every rule was tested against a suite of historical DeFi failures using only information available *before* each incident — no retroactive scoring.
- **Adversarial review.** Each proposed rule survived dedicated adversarial passes, including full sweeps of the live registry to hunt for false-positive damage. Rules that fired on positions they shouldn't were rewritten or rejected.
- **A standing acceptance test.** The current ruleset catches 21 of 23 in-scope loss events with **zero false positives** — the control incidents (blue-chip failures that no honest ex-ante rule should catch, like USDC during the SVB weekend) remain untouched. Every future policy version must re-run this suite before ratification; a change that fires on the controls is malformed by definition.
- **Amendment discipline.** Every policy version carries a changelog tracing each change to its evidence; superseded versions are archived unmodified, never overwritten; material changes flow through the same proposal → adversarial review → ratification pipeline as Hallmark methodology amendments.
- **An annual performance study.** ForgeYields commits to publishing a recurring review measuring how scores and verdicts performed against realized outcomes.

## Where to go next

- [Methodology →](methodology.md) — how Hallmark computes the scores this policy consumes
- [Transparency & public scores →](transparency.md) — the published artifacts and how to verify them
- [How a score is built →](example-score.md) — a worked example, end to end
