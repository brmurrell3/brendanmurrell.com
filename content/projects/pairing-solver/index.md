---
title: "pairing-solver"
description: "Finding the Lowest Strategy-Based Margin for an Options Book"
weight: 6
tags: ["optimization", "options", "margin", "cpp", "linear-programming"]
summary: "C++20 solver that splits an options book into FINRA and Cboe margin structures at the lowest total requirement."
ShowToc: false
---

[Source code](https://github.com/brmurrell3/pairing-solver) ·
[Proofs](https://github.com/brmurrell3/pairing-solver/blob/main/docs/proofs.md) ·
[Benchmark tables](https://github.com/brmurrell3/pairing-solver/blob/main/docs/benchmarks.md)

An account that holds options must keep margin against them. FINRA Rule
4210(f)(2) and Cboe Rule 10.3 let offsetting positions be margined together as
spreads, straddles, covered calls, collars and boxes, each at a lower
requirement than its legs held alone. The rules say which structures are
allowed, but not how to split a book into them. The same book can usually be
split many ways, with different totals.

pairing-solver finds the split with the lowest total requirement and reports
it leg by leg. It is a C++20 library with its own simplex and branch-and-bound
solver, built only from public rule texts and published papers.

## One group instead of many spreads

The usual approach matches legs into named spreads one at a time. The rules
also allow any balanced set of calls and puts on one underlying to be
margined as a single group at its maximum possible loss. That is never worse,
and often much better:

<!-- The diagram is drawn for a white background, so it keeps a white card in dark mode; the link opens it full size for small screens. -->
<a href="condor.svg" style="display:block;box-shadow:none"><img src="condor.svg" alt="Four option legs: long put 90, short put 95, short call 105, long call 110. Margined as two separate vertical spreads they cost $500 each, $1,000 in total. Margined as one group they cost $500, because the payoff at expiration, drawn below, never falls below minus $500: the put side and the call side cannot both lose. The payoff is flat outside the strikes and bends only at 90, 95, 105 and 110, which are marked as the checked points." style="width:100%;padding:10px 12px;background:#fff;border-radius:var(--radius)"></a>

Three facts make this a small optimization problem, each proved in
[docs/proofs.md](https://github.com/brmurrell3/pairing-solver/blob/main/docs/proofs.md):

- **Check only the strikes.** A group's payoff bends only at its strikes, so
  its maximum loss is the worst of one linear row per strike.
- **One group per class.** Merging two valid groups gives a valid group that
  never costs more, so one group per underlying, exercise style and listing
  is enough.
- **No pairing variables.** Every short must pair with a long that expires no
  earlier. That reduces to one inequality per expiration instead of one
  variable per possible pair.

Stock, straddles, conversions and boxes are added as fixed templates priced by
the rule engine. The result is a small integer program over contract
quantities, and leaving every position standalone is always a valid answer.

## The solver

<a href="flow.svg" style="display:block;box-shadow:none"><img src="flow.svg" alt="Two rows of steps. Load a book, once per book: positions and rule set, then the model, then dual simplex and branch-and-bound, then an independent rule checker, then structures and requirement. Price an order, before each order is routed: the order, then changing bounds or rebuilding the model, then re-solving from the saved basis, then the independent rule checker, then the requirement after the order. A dashed arrow shows the basis, factorization and pairing kept from loading the book and reused for each order." loading="lazy" style="width:100%;padding:10px 12px;background:#fff;border-radius:var(--radius)"></a>

- **Dual simplex** with a sparse LU basis factorization. A resize changes
  only bounds, so the book's last basis stays valid and the re-solve starts
  from it. Other orders rebuild the model, starting from the same basis.
- **Branch-and-bound** stops within 1% of its own proven lower bound by
  default, and runs exact on small books.
- **An independent checker** re-derives every structure's requirement from the
  rules, so the reported total never depends on the solver's arithmetic.

## Results

On generated books, against [HiGHS](https://highs.dev) as the reference
optimum and against a greedy pairing baseline:

| Check | Result |
|---|---|
| 3,000 small portfolios against exhaustive search | Same optimum on every one |
| 1,000 portfolios of up to 40 series against HiGHS | Within 1%, never below |
| Against greedy pairing, retail-like books | 5.6% lower requirement on average |
| Against greedy pairing, market-maker books | 11.0% lower |
| Against greedy pairing, adjusted deliverables | 16.5% lower |
| Pricing one order on a 1,000-series book | 10 to 20 ms median, all 240 under 50 ms |
| Loading a 1,000-series book | 0.42 s |

Times are from an Apple M3 Max. The
[benchmark tables](https://github.com/brmurrell3/pairing-solver/blob/main/docs/benchmarks.md)
have the full runs and the command that regenerates them.
