# tiny-chess-01 — a chess engine in 4 KB of Python

A complete chess engine in one Python file. `main.py` (4,053 B, cap
4,096) is the kit-`makefun compress` packing of a readable source; the
tracked snapshot stays under its 8,192 B cap. It plays via the
environment's HTTP API: reads `MAKEFUN_ENV_URL`, polls `/game`, answers
each position with a UCI move posted to `/move`.

## How it plays

- **Board**: 0x88 representation (128 squares; off-board = `& 0x88`),
  FEN parsing, incremental Zobrist hashing, make/unmake with a full undo
  stack.
- **Search**: negamax alpha-beta with a transposition table, check
  extensions, and principal-variation search (zero-window probes,
  re-searched on fail-high). The root runs iterative deepening, seeded
  with the previous iteration's best move.
- **Pruning (v2)**: null-move pruning at depth ≥ 3 (reduction 2, or 3 past
  depth 6; skipped in check, near mate, or without non-pawn material) and
  late-move reductions for quiet non-killer moves from the fourth legal
  move.
- **Quiescence**: captures-only search with stand-pat and delta pruning
  (v2): captures that cannot raise alpha by victim value + 200 are skipped
  unless in check or promoting.
- **Ordering**: transposition-table move first, then MVV-LVA captures, then
  two killer moves per ply.
- **Evaluation**: material (100/320/330/500/900) plus piece-square tables,
  tapered between middlegame/endgame king tables by remaining non-pawn
  material, plus a bishop-pair bonus. Repetition and the fifty-move rule
  score as draws in search.
- **Budget**: a fixed 80,000 nodes per move — never a clock — so every game
  replays move for move. If the budget runs out mid-iteration, the last
  completed iteration's move stands.

## Running it

```
MAKEFUN_ENV_URL=<env url> python3 main.py
```

With no environment URL set, the entry exits immediately with
`MAKEFUN_ENV_URL unset`. Verified locally against a stub environment:
from the start position it answered `e2e4` after its node-budget search
and exited cleanly on `done`.

## v1 → v2 changes

- Added null-move pruning, late-move reductions, and quiescence delta
  pruning.
- Root ordering seeded with the previous iteration's best move (see
  Search).
- Removed a dead adaptive-budget path that could never trigger in a
  trial.
- Rejected on measurement: a passed-pawn bonus (−197 Elo in screening,
  ~35% slower per node).

## Measured practice results

Official practice gauntlets, 8 games per trial vs the house anchors
(sunfish-4k practice mean: 1325):

| trial | v1 Elo (W/D/L) | v2 Elo (W/D/L) |
|---|---|---|
| 0 | 1326 (3/1/4) | 1669 (5/2/1) |
| 1 | 1464 (4/1/3) | 1530 (5/0/3) |
| 2 | 1530 (4/2/2) | 1253 (3/0/5) |
| mean | 1440 | **1484** |

Per-trial variance is ±300 Elo and trial 2 is a real regression on its
openings; the +44 mean edge is real on average, not a lock. An
independent-opening screen at equal 30k-node budget scored v2 +237 Elo
over v1.
