---
title: "tt_um_brmurrell3_m31_accel"
description: "Mersenne-31 Arithmetic Accelerator"
weight: 3
tags: ["hardware", "verilog", "formal-verification", "rust", "zk"]
summary: "Verilog RTL and formal verification for a Mersenne-31 modular arithmetic ASIC, taped out on Tiny Tapeout in IHP 130 nm."
aliases: ["/projects/m31-accel/"]
---

[RTL and formal checks](https://github.com/brmurrell3/tt_um_brmurrell3_m31_accel) ·
[Chip page on Tiny Tapeout](https://tinytapeout.com/chips/ttihp26a/tt_um_brmurrell3_m31_accel) ·
[Host stack: m31-accel](https://github.com/brmurrell3/m31-accel)

M31-ACCEL is a small ASIC that adds, subtracts and multiplies numbers modulo
the Mersenne prime 2^31 - 1. That field is the base field of Plonky3 and other
STARK provers. I designed the chip, checked the RTL formally, and wrote the
software that drives it from a proof system. It was taped out on the Tiny
Tapeout IHP 26a shuttle in IHP's 130 nm process. Everything below is verified
in simulation and by formal checks.

## Why this field

Reducing modulo a Mersenne prime needs no division. Because 2^31 is congruent
to 1, the top 31 bits of a product fold back onto the bottom 31 bits with one
addition, followed by one conditional subtraction. A couple of adders and a mux
replace a general modular reduction.

## What is on the chip

- Three 32-bit registers: an accumulator A and two operands B and C.
- ADD and SUB in one clock cycle.
- MUL and MAC (multiply-accumulate, A + B x C) in 32 cycles: 31 shift-and-add
  steps and one reduction step.
- An 8-bit bus, so a 32-bit value takes four loads or four reads. A BUSY pin
  is high while a multiply runs.
- A 66 MHz target clock, in a 1x2 Tiny Tapeout tile at 58% utilization.

## Verification

- **Formal:** 22 assertions and 4 reachability covers, run with SymbiYosys and
  Z3. This is bounded model checking to depth 42, not an unbounded proof. The assertions prove every result stays
  inside the field. They do not state the product itself, which simulation
  covers.
- **RTL simulation:** 32 cycle-accurate tests in Verilator.
- **Above the chip:** driver unit tests, a simulator that runs the real
  firmware against the Verilator model, and 118 Plonky3 tests that generate
  and verify STARK proofs through the driver.

## The host stack

The software that drives the chip lives in a second repository,
[m31-accel](https://github.com/brmurrell3/m31-accel):

- **RP2040 firmware in C.** It takes commands over USB serial and drives the
  chip's pins directly.
- **Rust driver.** It batches operations and talks to the firmware through a
  transport trait, so the same code runs against the real chip, the simulator
  or a mock. It contains no unsafe code.
- **Plonky3 backend.** A fork of Plonky3 routes dot products of eight or more
  elements to the chip.

## Limits

One serial multiplier behind an 8-bit bus is a way to learn the whole path from
RTL to a proof system. It is not a way to speed up a prover. The same
arithmetic core, driver and integration would carry over to an FPGA or a
larger ASIC with pipelined multipliers and a faster link.
