---
title: "M31-ACCEL"
date: 2026-02-01
tags: ["rust", "hardware", "zk", "verilog", "formal-verification"]
summary: "Custom ASIC for modular arithmetic over GF(2^31 - 1), targeting ZK-STARK proof acceleration."
---

**[GitHub](https://github.com/brmurrell3/m31-accel)**

A custom ASIC for modular arithmetic over GF(2^31 - 1) — the base field of Plonky3 and other STARK provers. The Mersenne prime structure enables bit-folding reduction: a couple of adders and a mux replace division-based modular arithmetic.

### Full stack

The project implements and tests from silicon to proof system:

- **RTL** (Verilog) — ADD, SUB, MUL, MAC operations with a 31-cycle shift-and-add multiplier
- **RP2040 firmware** (C) — command dispatch and GPIO bit-bang interface
- **Rust host driver** — batch builder with `Transport` trait abstraction, `#![deny(unsafe_code)]`
- **Plonky3 integration** — dot product override for ASIC-accelerated STARK proving

### Verification

- 22 formal RTL assertions + 4 reachability covers (SymbiYosys/Z3)
- 201+ tests across 5 layers — RTL simulation, formal verification, driver unit/property/stress, Plonky3 integration, end-to-end bring-up
- Fabrication via [Tiny Tapeout](https://tinytapeout.com/) on IHP sg13g2 130nm
