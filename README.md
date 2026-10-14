<!-- solve-problem:2b96d6a3-1d56-4029-9f41-174a42933604 -->
# Liquid-staking collateral run feedback

A [Tangle Solve problem](https://solve.tangle.tools/problems/2b96d6a3-1d56-4029-9f41-174a42933604).

## What is the problem?

Build and analyze a game-theoretic/dynamic model of a **run on liquid-staking collateral** in a DeFi stack with three coupled layers:

1. **Staking layer.** A liquid staking pool (LSP) stakes capital C with validators and issues receipt tokens (e.g., stETH-style). Validators can be malicious or degraded; with endogenous probability the pool suffers slashing losses or rewards degradation (the "low-stake attack": a validator controlling a small fraction of stake extracts MEV or attacks consensus, imposing losses on the pool).
2. **Redemption layer.** Receipt-token holders exit either by selling on a secondary market (where the receipt trades at a discount d < 1 relative to the underlying) or by joining a redemption queue with withdrawal delay τ and limited withdrawal capacity per epoch.
3. **Credit layer.** Receipt tokens are posted as collateral in lending markets. When the secondary-market price falls enough (or an oracle marks the receipt at a discount), positions become undercollateralized and are liquidated — forcing sales of receipt tokens, which widens the discount further.

Simultaneously, **short-sellers** can pre-position: borrow the receipt token, short it, and (if adversarial) trigger or amplify validator attacks to deepen the discount. The problem: characterize the joint dynamics of (d, queue length, liquidation cascade size, validator attack incentive), and determine equilibrium conditions — when is the stable "no-run" equilibrium sustained, and when does a self-fulfilling run occur? Deliverables include conditions on pool parameters (fee structure, queue capacity, τ, unbonding period, collateral haircuts) that rule out profitable attack-triggered runs.

The original statement is informal ("model... under simultaneous short-selling"); the formalization above is the most reasonable reading. Ambiguity: whether agents are strategic atomistic depositors or a single large attacker, and whether time is discrete epochs or continuous.

## Why does it matter?

Liquid staking receipts are the dominant form of collateral in Ethereum DeFi (multibillion-dollar base), so a coupled validator-attack → discount → liquidation-cascade loop is a systemic-risk channel that no current mechanism explicitly prices. A solution would let LSPs set withdrawal-queue and fee parameters and lending markets set haircuts that provably deter leveraged attacks, and would give regulators/risk teams a run-risk metric analogous to Diamond–Dybvig for banks. It also connects validator-level cryptoeconomics to credit markets, a link currently modeled only piecemeal.

## What's already known?

- **Diamond–Dybvig / bank-run theory:** first-come-first-served redemption with illiquid underlying yields multiple equilibria including runs; withdrawal queues/delays are the standard mitigation. Directly analogous to LSP unbonding queues.
- **Leveraged-attack and manipulation literature on AMMs/lending:** known results that attackers can profit from price manipulation when their payoff in a downstream market (loans, futures) exceeds the cost of moving the spot price — the natural template here, with the "spot market" replaced by the receipt discount.
- **Liquidation-cascade models:** discrete/continuous models of cascading liquidations in lending markets show feedback amplification when collateral and debt assets coincide.
- **The cited key work** (Yang, Yaish, Gervais, Zhang 2026, arXiv:2605.01025) reportedly analyzes low-stake attacks on liquid staking pools — an attacker with a minority of stake extracts value at the pool's expense — but per the census it does **not** model the credit/short-selling feedback; that extension is the open problem. **I cannot verify this citation or its contents; treat its exact results as unconfirmed.**
- Gap: no existing work, to my knowledge, combines all three layers (validator attack incentive, redemption-queue run dynamics, leveraged short liquidation feedback) in one equilibrium model.

## What would count as solving it?

1. **Formal model** with explicit strategy spaces (depositors: hold/redeem/sell; attacker-validator: attack/not; short-seller: pre-position, size, trigger), timing, and price/queue update rules.
2. **Equilibrium characterization:** prove existence and identify the threshold condition (on attack cost, short size, haircut, queue capacity, τ) separating (a) a unique no-run equilibrium from (b) an equilibrium in which a profitable attack-plus-short run occurs. A closed-form or tight parametric threshold is ideal.
3. **Mechanism design corollary:** show a concrete parameter choice (e.g., dynamic withdrawal fee, haircut schedule) eliminates the attacker's profit for all short positions below a bound — with a proof, not simulation.
4. Alternatively/additionally: a lower bound showing some feedback profit is unavoidable in any queue-based design (impossibility), or a calibrated simulation study demonstrating realistic parameter regimes with welfare quantification.

## How to start

1. Write down the minimal two-agent model first: one attacker-validator choosing attack effort a with cost κ(a) inducing a slash/loss L(a) on the pool; one short-seller with position s facing borrow cost; one representative depositor run decision via a Diamond–Dybvig-style utility comparison with queue discipline. Solve for the attacker's break-even condition ignoring liquidations — this is the "no-feedback" baseline.
2. Add the lending layer: a standard overcollateralized loan with liquidation threshold triggered by the receipt's secondary-market price, where the short-seller's exit trade moves the price (constant-product AMM or linear impact). Compute the attacker+short joint profit; show when it exceeds the no-feedback baseline — this isolates the *feedback gain*, the paper's core object.
3. Read Diamond–Dybvig (run equilibria and suspension mechanisms) and the AMM-manipulation/leveraged-attack literature for the single-block manipulation profit condition; then the cited Yang–Yaish–Gervais–Zhang paper for the validator-side attack economics (verify it exists first).
4. Main obstacle: tractability of the fixed point "discount depends on expected queue length, queue length depends on expected discount," plus the attacker's endogenous effect on fundamentals. Start with linear impact and a two-period model; only then generalize.

## Key references

- Diamond, D. and Dybvig, P. (1983), "Bank Runs, Deposit Insurance, and Liquidity," *Journal of Political Economy* 91(3). Canonical model of run equilibria under first-come-first-served redemption and suspension/queue remedies — the structural template for LSP redemption dynamics.
- Yang, Yaish, Gervais and Zhang (2026), "Your Loss is My Gain: Low Stake Attacks on Liquid Staking Pools," arXiv:2605.01025. Per the census, analyzes low-stake validator attacks on liquid staking pools; unverified citation — confirm before relying on it.
- Milne, A. (2009), "The Fall of the House of Credit" and Brunnermeier, M. (2009), "Deciphering the 2007-08 Liquidity and Credit Crunch" (*J. Econ. Perspectives*). Non-academic alternatives: for fire-sale/liquidity-spiral feedback structure, Brunnermeier and Pedersen (2009), "Market Liquidity and Funding Liquidity," *Review of Financial Studies* 22(6), is the right reference — models the margin spiral where falling collateral prices trigger liquidations that further depress prices.
- For the mechanism-design side of DeFi runs: a survey of stablecoin/depeg run models, e.g., Klages-Mundt et al. (2020), "Economic Attack Vectors on DeFi," arXiv:2002.02499 (verify relevance). AMM manipulation profit conditions: see references therein on oracle/price manipulation attacks (e.g., the "ElasticSwap" and Curve-style manipulation analyses). (Several specific attack-paper citations here should be verified.)

## Warnings

- The anchor citation (arXiv:2605.01025) is dated 2026 and could not be verified against the real literature; do not assume its results — re-derive the low-stake attack economics from first principles if it is unavailable.
- Common mistake: treating the receipt discount as exogenous. The whole point is that it is jointly determined by the run, the short, and the liquidation cascade — a fixed point, not an input.
- Dead end to avoid: full continuous-time stochastic modeling of all three layers at once; it is intractable and referees will want the discrete/linear-impact version proved first.
- Subtlety: oracle design matters enormously. If the lending market's oracle uses the redemption-queue-adjusted price vs. the spot AMM price, cascade thresholds differ; state your oracle assumption explicitly.
- Distinguish "attack profitable ex post" (one-shot manipulation, largely understood) from "attack profitable in expectation with strategic depositors who may run" (the genuinely open part).

---
*`BCGT-000348` · flagged for review*

## Acceptance guidance

1. **Formal model** with explicit strategy spaces (depositors: hold/redeem/sell; attacker-validator: attack/not; short-seller: pre-position, size, trigger), timing, and price/queue update rules.
2. **Equilibrium characterization:** prove existence and identify the threshold condition (on attack cost, short size, haircut, queue capacity, τ) separating (a) a unique no-run equilibrium from (b) an equilibrium in which a profitable attack-plus-short run occurs. A closed-form or tight parametric threshold is ideal.
3. **Mechanism design corollary:** show a concrete parameter choice (e.g., dynamic withdrawal fee, haircut schedule) eliminates the attacker's profit for all short positions below a bound — with a proof, not simulation.
4. Alternatively/additionally: a lower bound showing some feedback profit is unavoidable in any queue-based design (impossibility), or a calibrated simulation study demonstrating realistic parameter regimes with welfare quantification.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/2b96d6a3-1d56-4029-9f41-174a42933604?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000348.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000348)
