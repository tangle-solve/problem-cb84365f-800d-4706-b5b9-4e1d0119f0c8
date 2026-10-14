<!-- solve-problem:cb84365f-800d-4706-b5b9-4e1d0119f0c8 -->
# Incentives for accurate dependency declarations

A [Tangle Solve problem](https://solve.tangle.tools/problems/cb84365f-800d-4706-b5b9-4e1d0119f0c8).

## What is the problem?

In blockchains with parallel transaction execution (e.g., proposed Ethereum/Monad-style designs), each transaction declares which pieces of state it will read and write — its **access list** — so that a scheduler can partition non-conflicting transactions across threads. The declaration is supplied by the transaction sender, who may be strategic. The problem: design a fee market / mechanism that **elicits truthful access declarations** when the true access set may itself be state-dependent (known fully only after execution, or privately known to the sender).

Formalize as follows. There are $n$ transactions; transaction $i$'s sender submits a declaration $d_i \subseteq \mathcal{K}$ (a set of state keys or objects). The true access set $A_i \subseteq \mathcal{K}$ is revealed at execution. A scheduler runs transactions in parallel when their declared (or actual) sets are disjoint; conflicts cause aborts/re-execution, incurring latency cost. The mechanism charges each sender a fee $p_i(d_i, A_i, \text{outcome})$ and possibly aborts. Desired properties:

1. **Truthful elicitation is (weakly) dominant**: for any distribution over state and other transactions, declaring $d_i = A_i$ (or one's best estimate) maximizes expected utility. This generalizes **strictly proper scoring rules** to set-valued, state-dependent quantities.
2. **No overstatement incentive**: inflating $d_i$ must not reduce expected cost (overstatement destroys parallelism — it forces spurious serialization).
3. **No understatement/concealment incentive**: shrinking $d_i$ must not reduce expected cost (understatement causes conflicts, aborts, wasted work).
4. The mechanism should remain budget-reasonable, Sybil-resistant, and compatible with base-fee (EIP-1559-style) markets.

Ambiguity to note: whether $A_i$ is (a) known deterministically to the sender, (b) a random variable with sender-known distribution (elicitation of beliefs), or (c) adversarially concealable post-hoc. Variant (b) is the most tractable and most defensible formalization.

## Why does it matter?

Parallel execution is the main route to scaling smart-contract chains beyond single-threaded EVM throughput, and every proposed design (Monad, Polygon Async, Parallel EVM proposals, the 2025 multiresource fee literature) consumes declared access lists. If senders can profit by misdeclaring — e.g., declaring fewer keys to pay lower "footprint" fees — conflict rates explode and parallelism collapses to serial re-execution, voiding the performance gains. A truthful mechanism would also yield clean theoretical foundations for parallel-gas fee markets, analogous to what EIP-1559 did for base-fee design. Conversely, if it stays open, production designs must fall back on conservative estimates or full-state locking, sacrificing throughput.

## What's already known?

- **Acilan et al. (2025)** propose a transaction fee market for parallel execution with access-list-based pricing, but assume declared access sets are accurate or conservatively bounded; they do not analyze strategic misreporting. (verify the exact assumption — this is the paper to read first for the fee model.)
- **Proper scoring rules** (Gneiting & Raftery 2007) give truthful elicitation of probability forecasts; extensions exist for sets/estimation (e.g., report-precision and "property-based elicitation" of Lambert et al.), but none handle the two-sided externality here: one sender's declaration changes *others'* execution costs via scheduling.
- **Strategyproof scheduling with private processing times / requirements** (classic OR literature; also recent blockchain scheduling papers) shows truthful scheduling with machines and deadlines is largely impossible without payments that violate budget balance (related to Myerson–Satterthwaite-style impossibilities). The multi-dimensional type (an arbitrary subset of a huge key space) makes standard VCG-style remedies computationally infeasible and Sybil-fragile.
- Practitioner designs (Monad's optimistic parallel execution with static access sets) dodge the problem via re-execution penalties — effectively charging for *detected* understatement — but this punishes only stochastic detection, not overstatement, and treats state-dependent conflicts as unavoidable noise rather than an elicitation failure. No formal incentive analysis exists. (verify)

Net: no published mechanism is known to be truthful for set-valued, state-dependent access declarations under Sybil attacks; no impossibility result rules it out either.

## What would count as solving it?

Any of the following, in increasing strength:

1. **Impossibility result**: a proof that no mechanism (with reasonable budget/Sybil constraints) makes truthful access declaration dominant when types are subsets of a key space and utilities depend on parallelism — this would itself resolve the field's direction.
2. **Positive mechanism**: a fee rule $p_i(d_i, A_i)$ under which truth-telling is dominant (or Bayesian incentive compatible given sender beliefs about $A_i$), with overstatement and understatement each strictly dominated, plus an analysis of revenue/budget and Sybil-resistance. Matching upper/lower bounds on the "penalty needed" would be ideal.
3. **Weaker but publishable**: a mechanism that is $\varepsilon$-truthful or truthful only for a restricted class (deterministic $A_i$, or bounded set size), with an experiment showing conflict rates stay near the truthful baseline under best-response dynamics.

A referee would want: precise game form, equilibrium concept, an explicit counterexample showing naive schemes (flat per-key fees, abort penalties only) fail, and the formal guarantee.

## How to start

1. Read Acilan et al. (2025) §'s on the fee model to fix notation for how access declarations currently enter pricing — this defines the design space you must stay compatible with.
2. Write down the two "straw man" mechanisms and find their exploits: (a) charge $\propto |d_i|$ (understatement pays off); (b) charge $\propto |A_i \setminus d_i|$ on conflict (overstatement is free, destroying parallelism). These exploits delimit the design space and are likely the core of any impossibility argument.
3. Try a scoring-rule construction: treat $d_i$ as a forecast of $A_i$, pay via a proper scoring rule evaluated at execution time, plus an externality term for scheduling cost imposed on others. Check whether the externality term reintroduces an overstatement incentive.
4. Main obstacle: the key space is exponential in size, so any per-key VCG computation is infeasible; and Sybil agents can split transactions to game set-based fees. Any positive result must be pairwise/local (per-conflict pricing) rather than global optimization.

## Key references

- Acilan, C., et al. (2025). *Transaction Fee Market Design for Parallel Execution*. arXiv:2502.11964. Proposes fee markets pricing transactions by declared access lists for parallel-execution chains; assumes honest/conservative declarations, leaving the strategic question open. https://arxiv.org/abs/2502.11964
- Gneiting, T., & Raftery, A. E. (2007). *Strictly proper scoring rules, prediction, and estimation*. JASA 102(477). The canonical theory of truthful elicitation of forecasts; the natural technical toolkit for treating declarations as predictions of actual access sets.
- Lambert, N., Pennock, D., & Shoham, Y. (2008). *Eliciting properties of probability distributions*. EC '08. Generalizes proper scoring rules to eliciting statistics/properties, relevant for set-valued reports. (verify venue/details)
- Related literature on strategyproof scheduling with private processing times (classic scheduling-mechanism papers; e.g., work deriving from Dutta & others on scheduling mechanisms with payments) — establishes impossibility/truthfulness frontiers when agents' types are private resource requirements. (verify — no single canonical citation identified in the census record.)

## Warnings

- The census record is sparse; the "key works" entry (Acilan et al.) is the only verified anchor — read it before trusting anything above about the 2025 literature.
- Distinguish carefully between the sender knowing $A_i$ deterministically vs. only its distribution: the truthful mechanism differs (dominant-strategy vs. proper-scoring-rule Bayesian), and conflating them is the most common way to write a false proof.
- Aborts are observable but the *counterfactual* access set of an aborted transaction may not be — a mechanism cannot charge for keys a transaction *would have* touched, which breaks naive scoring rules.
- Overstatement is the failure mode most real designs ignore; a mechanism that only penalizes detected understatement is not a solution.
- Sybil attacks (splitting one transaction into many with disjoint declarations) can defeat set-cardinality fees; address or explicitly exclude.

---
*`BCGT-000289` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:

1. **Impossibility result**: a proof that no mechanism (with reasonable budget/Sybil constraints) makes truthful access declaration dominant when types are subsets of a key space and utilities depend on parallelism — this would itself resolve the field's direction.
2. **Positive mechanism**: a fee rule $p_i(d_i, A_i)$ under which truth-telling is dominant (or Bayesian incentive compatible given sender beliefs about $A_i$), with overstatement and understatement each strictly dominated, plus an analysis of revenue/budget and Sybil-resistance. Matching upper/lower bounds on the "penalty needed" would be ideal.
3. **Weaker but publishable**: a mechanism that is $\varepsilon$-truthful or truthful only for a restricted class (deterministic $A_i$, or bounded set size), with an experiment showing conflict rates stay near the truthful baseline under best-response dynamics.

A referee would want: precise game form, equilibrium concept, an explicit counterexample showing naive schemes (flat per-key fees, abort penalties only) fail, and the formal guarantee.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/cb84365f-800d-4706-b5b9-4e1d0119f0c8?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000289.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000289)
