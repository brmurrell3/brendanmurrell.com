---
title: "pairing-solver"
description: "Finding the Lowest Strategy-Based Margin for an Options Book"
weight: 6
tags: ["optimization", "options", "margin", "cpp", "linear-programming"]
summary: "C++20 solver that splits an options book into FINRA and Cboe margin structures at the lowest total requirement."
---

[Source code](https://github.com/brmurrell3/pairing-solver) ·
[Proofs](https://github.com/brmurrell3/pairing-solver/blob/main/docs/proofs.md) ·
[Benchmark tables](https://github.com/brmurrell3/pairing-solver/blob/main/docs/benchmarks.md)

An account that holds options must keep margin against them. U.S. rules
(FINRA Rule 4210(f)(2) and Cboe Rule 10.3) let positions that offset each
other be margined together as spreads, straddles, covered calls, conversions,
collars and boxes, and each of those structures carries a lower requirement
than its legs held alone. The rules say which structures are allowed. They do
not say how to find the cheapest way to split a book into them, and the same
book can usually be split many ways with different totals.

Many systems pair greedily: take the most obvious spread, then the next. On
seeded test books, greedy pairing left the requirement 5% higher than
necessary on retail-like books, 11% on market-maker books, and 15% when some
contracts deliver 150 or 10 shares after an adjustment.

pairing-solver finds the lowest requirement the rules allow, and reports the
structures leg by leg. It is a C++20 library with a small C interface, built
only from public rule texts and published papers, with its own simplex and
branch-and-bound solver.

## The formulation

The model replaces a long list of named spreads with one flexible group per
class of options, plus fixed templates for the structures that involve stock
or straddles.

- **Max-loss group.** Any set of calls and puts on one underlying is a valid
  spread if long and short share-equivalents balance within calls and within
  puts, and each short can be paired with a long of the same type that expires
  on or after it. Its requirement is its maximum potential loss. Under the
  default expiration reading, one group per (underlying, exercise style,
  listing) class is enough: merging two valid groups gives a valid group, and
  maximum loss is subadditive, so merging never costs more.
- **Loss rows at strikes.** A balanced group's payoff is piecewise linear in
  the underlying price, with kinks only at strikes and flat tails, so its
  minimum is at a strike. One loss row per strike gives the exact maximum loss.
- **Suffix rows instead of flows.** Whether every short can be paired with a
  later-expiring long is a matching problem on a nested graph. It is feasible
  exactly when, for each expiration, the shorts expiring on or after it are
  covered by the longs expiring on or after it. That is one inequality per
  expiration in place of a flow variable per pair.
- **Templates.** Short straddles, covered positions, conversions, collars and
  long boxes are columns priced by the rule engine. Each column costs its
  requirement less the standalone requirements it replaces.

The loss rows have a simple structure. Sort the group's strikes
{{< math >}}K_1 < \dots < K_n{{< /math >}}, let {{< math >}}D_j{{< /math >}}
be the running signed quantity of all its options at strikes up to
{{< math >}}K_j{{< /math >}}, and let {{< math >}}Q_0{{< /math >}} be the sum
of signed put quantity times strike. The netted payoff at
{{< math >}}K_m{{< /math >}} is

{{< math display >}}
P(K_m) = Q_0 + \sum_{j<m} D_j \, (K_{j+1} - K_j)
{{< /math >}}

so every loss row is a prefix sum of one strike-ordered sequence, and all of
them can be evaluated in one pass.

The result is a small integer program. The variables are contract quantities,
each bounded by what the account holds, and leaving every position standalone
is always feasible, so every solve has an answer.

## The solver

I wrote the solver from scratch. HiGHS appears only in the tests, as an
independent source of proven optima.

- **Dual simplex.** A bounded-variable dual simplex sized for one
  (account, underlying) subproblem, with dual steepest-edge pricing. Accounts
  that hold the same series share the constraint matrix and differ only in
  bounds, so one account's basis stays dual feasible for the next.
- **Branch-and-bound.** Best-first search with plunging and a rounding dive.
  Every search has a node limit and reports its status and proven gap when it
  stops. By default it stops within 1% of its own bound; small books run exact.
- **Share-size congruences.** Mixing 100-, 150- and 10-share contracts in one
  balance row creates whole-lot conditions the LP relaxation cannot see. One
  integer column per share size, branched on first, exposes them.
- **Compact straddles.** One template per short put and short call pair grows
  as puts times calls: 57,600 of 58,601 columns on a 1,000-series book.
  Ranking the shorts by their uncovered requirement turns straddle pairing into
  a nested transportation problem, so two columns per short and one row per
  rank give the same optimum with 1,765 columns in total.

An independent checker re-derives every structure's validity and requirement
from the rules, separately from the model builder, and confirms that each
position is used exactly. The reported total never depends on the LP's
arithmetic.

## Proofs

Every structural claim the model relies on has a written proof in
[docs/proofs.md](https://github.com/brmurrell3/pairing-solver/blob/main/docs/proofs.md)
and a randomized check in the tests.

- **Proved:** loss rows at the strikes are exact; the strike-walk formula
  above; the suffix rows are equivalent to a pairing flow (Hall's condition on
  a nested graph); one group per class suffices under the default expiration
  reading; the compact straddle form has the same optimum as one template per
  pair.
- **Integrality is not free.** The continuous optimum is not always
  whole-contract. Two counterexamples are kept as tests: three calls on one
  deliverable, where the LP is 0.67% below the integer optimum, and a one-lot
  iron condor at 0.37%. No two-series book in a search of 78,000 had a gap.
- **A dynamic program over strikes** solves the one-group, one-expiration case
  exactly. Its running time is pseudo-polynomial in contracts held and in the
  strike range over the tick. It was slower than branch-and-bound at every size
  I measured, so it stays in the repo as an independent exact check rather
  than as the solver.

## Results

All numbers come from seeded, generated books, not real accounts.

| Check | Books | Result |
|---|---|---|
| Against exhaustive search, exact mode | 3,000 portfolios of up to 8 positions | Same optimum on every one |
| Against HiGHS (`mip_rel_gap = 0`) | 1,000 portfolios of up to 40 series | Under 1% above the optimum, never below |
| Large cases against HiGHS | 5 books of 50 to 200 series | All within 0.9% |

| Book family | Books | Greedy saves | Solver saves | Greedy above solver (mean) |
|---|--:|--:|--:|--:|
| Retail | 496 | 55.4% | 57.9% | 5.4% |
| Adjusted deliverables | 496 | 37.6% | 47.8% | 15.5% |
| Market maker | 200 | 34.5% | 41.4% | 10.8% |

Savings are relative to margining every position on its own. The greedy
baseline is my own, and I made it stronger than a naive one: it recognizes
iron condors and covers the costliest shorts first, so the comparison is not
flattering to the solver.

## Performance is in progress

The next goal is to price a single order against an already-solved book of
about 1,000 positions fast enough to run before the order is routed. The
solver does not meet that yet. A book now keeps its solved state, so a resize
only moves bounds and re-solves from the previous basis, and the remaining
time is mostly in the dense basis inverse. The next step is a sparse LU
factorization with Forrest-Tomlin updates. Current measurements are in the
[benchmark tables](https://github.com/brmurrell3/pairing-solver/blob/main/docs/benchmarks.md).
