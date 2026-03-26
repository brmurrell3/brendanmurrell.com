---
title: "Q-View"
date: 2026-01-01
tags: ["go", "options", "volatility", "risk-neutral-density"]
summary: "Extracts and visualizes risk-neutral probability densities from live option prices using Breeden-Litzenberger."
---

**[GitHub](https://github.com/brmurrell3/q-view)**

Q-View extracts and visualizes risk-neutral probability densities from option prices using the Breeden-Litzenberger method. It fetches live options data from the Schwab API and Treasury rates from FRED.

### Features

- **Breeden-Litzenberger** density extraction from option chains
- **SVI fitting** for volatility surface parameterization
- **Black-Scholes** pricing engine
- **Live data** via Schwab API (options) and FRED API (Treasury rates)
- **Desktop GUI** built with Gio
- Built in **Go** — run with `--mock` for bundled test data, no API keys needed
