# ECE 180DA/DW — Problem Set 2 (Verilog practice)

**Thadeus Wang** · Fall 2026

## Status

Both problems complete and passing the provided testbenches (Verilator 5.020, `make all`):

```
=== p1  (rtl/) ===
P1 PASS  (3144 cycles checked, counter saturates correctly)
=== p2  (rtl/) ===
P2 PASS  (reset, r0, dual reads, write enable, bypass on/off)
```

Only `rtl/` was edited; `tb/`, `Makefile` and `hw_pkg.sv` are untouched. Build is lint-clean under the
homework's `-Werror` rules (LATCH, WIDTH, CASEINCOMPLETE, BLKSEQ, COMBDLY, MULTIDRIVEN).

## What I did

**P1 — overlapping `1101` detector (`rtl/p1_seq_detect.sv`)**
- Two-block FSM: an `always_comb` with default-hold (`state_d = state_q; found_o = 0;`) and a
  separate state register. Four states = the longest prefix of `1101` the stream currently ends
  with (none / `1` / `11` / `110`).
- The three transitions that matter: `S_1 --1--> S_11`, `S_11 --1--> S_11` (self-loop, the longest
  live prefix is still `11`), and `S_110 --1--> S_1` with `found_o = 1` — falling back to `S_1`
  rather than `S_IDLE` is what makes the detection overlapping (`1101101` gives two hits).
- `found_o` is a Mealy output (state AND current bit), so it is high during the cycle the 4th bit
  is presented. The counter increments on `found_o` and saturates at `8'hFF`
  (`else if (found_o && count_q != 8'hFF)`), so it reports "255, widen me" instead of wrapping.

**P2 — register file with bypass (`rtl/p2_regfile.sv`)**
- Write port: reset clears all eight entries with an unrolled `for` loop (eight parallel reset
  connections, not a loop in time); otherwise a write lands on the clock edge only if
  `wr_i.en && wr_i.addr != 0`.
- Read ports: combinational. `r0` is a constant zero (no flop behind it).
- Bypass: per read port, forward `wr_i.data` when `bypass_en_i && wr_i.en && wr_i.addr == rs_addr
  && rs_addr != 0`; otherwise return `regs_q[rs_addr]`. With bypass off a colliding read returns
  the old (stale) value, as the testbench requires. The cost is a comparator plus a mux on the
  already-critical read path — a cycle of latency traded for some delay.

---

# ECE180 Verilog Homework

Two problems, building on the lab (`WORKBOOK.md`).

**Start here: [`HOMEWORK.md`](HOMEWORK.md)**

## Quick start

```bash
verilator --version     # need 5.0+;  apt-get install verilator  /  brew install verilator
make all                # run both problems
```

## Submission

Graded for **completion**.

1. Push this directory to a GitHub repo and submit the URL.
2. Run `make all`, screenshot the `P1 / P2` lines, upload to Gradescope.

Stuck on one? Submit anyway, and note in your README what you tried and where it broke.

## Layout

```
HOMEWORK.md    the assignment — read this
Makefile
rtl/           you edit these
tb/            testbenches — do not edit
```

## Problems

| | Topic | File |
|---|---|---|
| 1 | FSM: overlapping "1101" detector + saturating counter | `rtl/p1_seq_detect.sv` |
| 2 | Register file: dual combinational reads, sequential write, bypass | `rtl/p2_regfile.sv` |

## Targets

| | |
|---|---|
| `make pN` | run problem N |
| `make all` | run both |
| `make waveN` | run and dump `build/pN/pN.vcd` |
| `make clean` | |
