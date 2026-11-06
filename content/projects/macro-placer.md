---
title: "Partcl/Hudson River Trading Macro Placement Challenge"
weight: 1
tags: ["python", "pytorch", "optimization", "eda", "placement"]
summary: "Chip macro placer I designed and implemented; ninth on the final leaderboard of the Partcl/HRT Macro Placement Challenge 2026 (verified mean proxy 0.9974)."
draft: false
---

**[GitHub](https://github.com/brmurrell3/macro-placer)**

Macro placement decides where a chip's few hundred macros, large fixed-size blocks such as memories, sit without overlapping, so that the wires between them stay short and the routing space does not crowd. The challenge scored 17 public IBM designs, within one hour per design, by a proxy cost of wirelength, placement density and routing congestion.

I designed and implemented Thinkorplace as lead author, with Xiaofei Zhang and Andrew Murrell advising. It moves all macros at once by gradient descent on a smooth stand-in for the score, legalizes the result, then runs exact coordinate descent that keeps a move only when the true score falls. An incremental evaluator makes that exact check about 1,000–18,000× cheaper than full recomputation.

It placed ninth on the final leaderboard of the Partcl/HRT Macro Placement Challenge 2026 (verified mean proxy 0.9974; the organizers report 132 verified eligible submissions), 31.6% below the published RePlAce baseline and 4.9% above the winner.

### Stack
- Python, PyTorch (CUDA and Apple-silicon GPUs), NumPy, SciPy

**[Read the write-up →](/research/thinkorplace/)** · [Technical report (PDF)](/research/thinkorplace/thinkorplace-technical-report.pdf)
