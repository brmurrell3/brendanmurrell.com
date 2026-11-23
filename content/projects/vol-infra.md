---
title: "Volatility Trading Infrastructure"
weight: 4
tags: ["rust", "python", "options", "volatility", "svi"]
summary: "Rust workspace for systematic options volatility: a parquet warehouse of SPX options, SVI surfaces and Greeks, and strategy screens that scan the surface for dislocations."
---

**[GitHub](https://github.com/brmurrell3/vol-infra)**

A Rust workspace for systematic options volatility. Options data (SPX, via /ES) flows into a parquet warehouse; SVI surfaces, Greeks, and realized vol are computed on top; strategy screens scan the surface for pricing dislocations and emit trade signals.

### Stack

- **vol-core -> vol-pricer -> vol-instruments -> vol-feed -> vol-engine** — types, pricing, instruments, live feed, and the engine that wires them together
- **vol-backtest** — strategy backtesting and attribution
- **vol-query** — DuckDB-backed warehouse queries
- **vol-sim** — surface and signal visualizer
- **vol-py** — Python bindings for research notebooks

### Scope

Pricing, feeds, warehouse processing, query tooling, and strategy screening are implemented and running against the warehouse. Broker execution is the next phase. The design targets hundreds to thousands of underlyings without a rework, not just SPX.
