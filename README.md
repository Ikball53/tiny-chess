# tiny-chess — make.fun entry (tiny-chess-01)

A chess engine in 4,096 bytes of Python. It plays the house gauntlet
(16 games, 8 rungs, anchors 855–1712) through the env over HTTP.

## The engine

`main.py` (4,074 bytes) is an LZMA + base85 packed engine; it unpacks and
execs itself, standard library only. No model call, no clock, no other file.

- 0x88 board, Zobrist hashing, transposition table, killer moves
- Alpha-beta with iterative deepening (max depth 10), quiescence search
  (captures + checks), classical piece-square-table evaluation with a
  tapered king endgame correction and a bishop-pair bonus
- Fixed node budget per move: 30,000 nodes (`CHESS_NODES` env override),
  so live and replay play the same moves (deterministic: fixed-seed
  Zobrist keys, no wall clock, no unseeded randomness)

## Interface

This competition's entry is a standalone driver, not an imported module:
`main.py` runs as `python3 main.py` with `MAKEFUN_ENV_URL` set and drives
the env (`POST /game`, `POST /move`). There is no `get_best_move` /
`get_name` import interface in the kit (the starter is the same shape).

## Verification

- Move generation: perft matches python-chess exactly on 7 standard
  positions (startpos, kiwipete, pos3–pos6, en-passant), depths 1–4,
  and 30 random playouts keep the incremental Zobrist key equal to a
  from-scratch recomputation. (Two perft expectations from an earlier
  draft — kiwipete 48/2039/97862, ep-check 6/89/1274 — are wrong for
  the FENs as written; python-chess gives 33/1494/52061 and 5/19/111,
  which is what this engine produces.)
- Speed (this VM, 1 core): 30,000 nodes in 0.83–1.16 s/move;
  ~19k nodes/s raw. Well inside the 10 s move limit and the 3,600 s
  trial budget.

## Practice-trial result (trial-0, 30,000 nodes/move)

Official scorer on the trial export: **Elo 1087 — 2 wins, 4 draws,
10 losses** (16 games, 866 s wall). All 10 losses by checkmate; all
4 draws by threefold repetition. Per rung: depth-2 1.5/2, depth-3 1/2,
depth-4 0.5/2, nodes-100k 0.5/2, nodes-250k 0.5/2, nodes-500k/1M/2M 0/2.
Full export: `work/trial_full/state/export/state.json`.

## Honest strength note

This engine is **weak against the house baseline**. The baseline
(sunfish 4k, NNUE eval, 50k nodes) rates ~1,360 mean on the practice
instances; this engine rates 1,087 — about 270 Elo below it. A prize
needs beating the baseline's mean with the 90% paired interval above 0
(roughly a ~90 Elo lead, i.e. ~1,450+), so on this evidence the entry
would not win a prize. The gap is structural: a classical PST eval
against sunfish's NNUE. Raising the node budget to 50,000 would gain
only ~30–50 Elo and would push the slowest move toward the 10 s limit
on the grader's half CPU, so the 30,000-node budget is kept.
