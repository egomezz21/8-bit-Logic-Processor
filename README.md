# 8-Bit Serial Logic Processor

A logic processor built in two stages: first as a 4-bit serial processor using discrete
digital logic ICs on a breadboard, then extended to an 8-bit version implemented in
SystemVerilog and verified on FPGA hardware using Vivado simulation and debug cores.


<img width="622" height="458" alt="Screenshot 2026-07-19 231348" src="https://github.com/user-attachments/assets/6931f86c-4f9e-44ed-b663-0d0e4af864ce" />


## Overview

This project designs a serial logic processor around four functional blocks — a register
unit, a computation unit, a routing unit, and a control unit — that together execute
bitwise logic operations one bit at a time over successive clock cycles.

The processor was first built and validated as a 4-bit design using discrete logic ICs
(NAND, NOR, XOR gates, D flip-flops, shift registers, and multiplexers) on a breadboard,
then extended to an 8-bit datapath and reimplemented in SystemVerilog, where it was
synthesized and verified directly on FPGA hardware.

## Architecture

- **Register Unit** — Two bidirectional shift registers (Register A, Register B) that hold
  operands and support hold, shift-right, and parallel-load modes. Register contents are
  displayed on LEDs for real-time observation.
- **Computation Unit** — Takes the serial bits of A and B plus a 3-bit function select
  (`F[2:0]`) and computes one of eight functions: **AND, OR, XOR, NAND, NOR, XNOR,
  SET, CLEAR**. Passes the unmodified A and B values through alongside the computed
  result.
- **Routing Unit** — Takes A, B, and the computed result `f(A,B)` and uses a 2-bit
  routing select (`R[1:0]`) to determine what gets written back to each register —
  update A, update B, leave a register unchanged, or **swap register contents**.
- **Control Unit** — A **Mealy finite state machine** with just two states (Reset,
  Shift/Hold) that sequences loading and the four shift cycles needed to complete a
  serial operation. The Mealy architecture was chosen specifically because it required
  fewer states than an equivalent Moore machine.

## Design Process

- Truth tables were built for all six core logic functions (AND, OR, XOR, NAND, NOR,
  XNOR), then converted into gate-level implementations using only the NAND, NOR, and
  XOR chips available in the lab kit (no dedicated AND/OR chips) — for example, an
  XOR-based invert circuit (`Y = A XOR C`) was used to derive NAND/NOR/XNOR from
  AND/OR/XOR without adding a second gate family.
- Next-state logic for the Mealy control FSM was derived using K-maps built directly
  from the control unit's state transition table.
- Considered a dedicated 4-bit counter IC to track the four required shift cycles, but
  chose to embed the counting behavior directly into the FSM's state variables instead —
  fewer ICs, less wiring, and easier synchronization between the datapath and control
  logic.

## Debugging Notes

During testing, the routing unit began behaving unpredictably due to resistors placed too
close together on the breadboard, causing unintended shorts. After correcting the
spacing, a second issue surfaced: the pull-down resistor for the R0 routing select line was
wired to the power rail instead of ground, causing incorrect routing behavior. Fixing the
resistor connection resolved the issue.

## FPGA / SystemVerilog Extension

The validated 4-bit design was extended to a full 8-bit datapath and reimplemented in
SystemVerilog, with the following module breakdown:

| Module | Purpose |
|---|---|
| `Processor` | Top-level module integrating all submodules and I/O |
| `register_unit` | Houses the two 8-bit shift registers (A, B) and their control signals |
| `Reg8` | Positive-edge-triggered 8-bit shift register with async reset, sync load |
| `compute` | Computes all 8 selectable logic functions on the current A/B bits |
| `router` | MUX-based routing of A, B, and `F_A_B` back into the registers |
| `control` | Sequential logic (FSM) generating shift/load control signals |
| `HexDriver` | Drives the FPGA board's seven-segment displays to show register values in hex |
| `sync_debounce` | Synchronizes and debounces mechanical pushbutton inputs to the FPGA clock |

**Verification:** The 8-bit design was verified through behavioral simulation, then
confirmed on real hardware using Vivado's Integrated Logic Analyzer (ILA) debug core. A
trigger was set on the Execute signal, and waveform traces of `Aval`, `Bval`, and `LED`
were captured to confirm correct operation — for example, a captured trace of `0xA6 XOR
0x7B` showed Register B correctly updating to `0xDD` while Register A remained
unchanged, confirming the computation and routing logic worked as designed.

## Tech Stack

- **Discrete stage:** NAND/NOR/XOR ICs, D-type flip-flops, bidirectional shift register
  ICs, multiplexers
- **FPGA stage:** SystemVerilog, Xilinx Vivado
- **Verification:** Behavioral simulation, Vivado Integrated Logic Analyzer (ILA)

## Timeline

January – February 2026
