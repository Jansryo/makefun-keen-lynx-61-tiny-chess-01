# tiny-chess-01 — agent.md

## Engine

`main.py` (4,021 bytes) is a 0x88-board chess engine: negamax with
alpha-beta pruning, iterative deepening on a FIXED node budget of 200,000
nodes per move (deterministic — no clock, no unseeded randomness),
transposition table (Zobrist hashing), quiescence over captures and
promotions (all evasions when in check), MVV-LVA + killer move ordering,
check extension, and incremental material + piece-square evaluation
(P100 N320 B330 R500 Q900, tempo +10, bishop-pair +30). It talks to the
tournament env over HTTP (`POST /game`, `/move` at `MAKEFUN_ENV_URL`)
and nothing else. Python 3.13 standard library only; it reads, imports
or runs no other file. Packed with the kit's sanctioned compressor:
`node cli/makefun.mjs compress dev.py --out main.py` (LZMA payload).

## Tests

Commands (from the repo root, kit env server local):

- `python3 test/trial.py <trial> main.py` — full 6-game practice trial
  vs the kit's house engines (anchors 855/1073/1199/1392/1497/1596).

Results (200k-node budget):

| trial | result | score | practice Elo |
|-------|--------|-------|--------------|
| trial-0 | 1W 1D 4L | 1.5 | 997 |
| trial-1 | 1W 3D 2L | 2.5 | 1,191 |
| trial-2 | 1W 1D 4L | 1.5 | 997 |

Mean Elo 1,062 across the three trials; the gate is 1,154 (1 of 3
trials above it). No timeouts at 200k: server-measured think max
7,557 ms, under the 10 s/move limit. (At 250k one move took 10.97 s
under VM throttling and lost a game on time, so the budget was cut
250k -> 200k.)

Move generation: ALL PERFT PASS (startpos d4=197281, kiwipete
d3=97862, ep-pins d4=43238); state test 1,200 makes, 0 failures;
python-chess move-set diff at depth 3, all 5 positions OK. The
minified source plays identically to the readable source: 7/7
positions, same best move and node count at 50k nodes.

## Size

`main.py`: 4,021 bytes (limit 4,096). Repo total under 8,192 bytes.
