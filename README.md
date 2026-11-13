# tiny-chess-01 entry

A chess engine in one Python file: 0x88 board, negamax alpha-beta with a
transposition table, quiescence, MVV-LVA/killer ordering and a
material + piece-square evaluation. It searches to a fixed node budget,
never a clock, so every trial replays move for move. `main.py` is packed
from a readable source with the kit's `makefun compress`.
