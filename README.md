# Chess Engine

A UCI-compatible chess engine written in C. It uses a 120-square mailbox board with piece lists, alpha-beta search, and a hand-tuned piece-square evaluation. It runs in any UCI GUI such as Arena.

## Features

- **Board representation:** 120-square mailbox with piece lists and Zobrist position hashing
- **Move generation:** full legal chess rules, including castling, en passant, and promotions, with captures-only generation for quiescence search
- **Search:**
  - Negamax alpha-beta with iterative deepening
  - Quiescence search
  - Principal variation table, used for PV-move ordering
  - MVV-LVA capture ordering
  - Killer moves and history heuristic
  - Repetition and fifty-move-rule detection
  - Time management (`wtime`, `btime`, `winc`, `binc`, `movestogo`, `movetime`) and `go infinite`
- **Evaluation:** material plus piece-square tables for pawns, knights, bishops, and rooks
- **Protocol:** UCI
- **Testing:** built-in perft for validating move generation

## Project Structure

| File | Purpose |
|---|---|
| `vice.c` | Entry point: initialises everything and starts the UCI loop |
| `uci.c` | UCI protocol: `position`, `go`, `isready`, `ucinewgame`, `quit` |
| `defs.h` | Shared types, constants, macros, and function prototypes |
| `init.c` | One-time initialisation of lookup tables and hash keys |
| `board.c` | FEN parsing, board printing, piece lists and material, board consistency checks |
| `bitboards.c` | Bit helpers (pop and count bits) |
| `hashkeys.c` | Zobrist position key generation |
| `data.c` | Global lookup tables |
| `attack.c` | Square-attacked detection |
| `movegen.c` | Move generation and move ordering scores |
| `makemove.c` | Make and take back moves |
| `search.c` | Alpha-beta, quiescence, iterative deepening |
| `pvtable.c` | Principal variation table |
| `evaluated.c` | Position evaluation (the tables in use) |
| `pst_tables.c` | Standalone copy of the piece-square tables (not in the build) |
| `perft.c` | Perft move-generation test |
| `io.c` | Move and square printing, move parsing |
| `validate.c` | Assertion helpers |
| `misc.c` | Timing and input polling |
| `arena/` | Arena GUI tournament settings |

## Building

You need a C compiler (GCC / MinGW).

```bash
make
```

Or compile directly:

```bash
gcc vice.c init.c bitboards.c hashkeys.c board.c data.c attack.c io.c movegen.c validate.c makemove.c perft.c search.c misc.c pvtable.c evaluated.c uci.c -o vice
```

This produces the `vice` executable (`vice.exe` on Windows). It has been developed and tested on Windows with MinGW-w64.

## Usage

### In a GUI

Add the executable as a UCI engine in Arena, Cute Chess, or any other UCI-compatible GUI.

### From the command line

```
$ ./vice
id name Vice 1.0
id author MaxLeo
uciok
isready
readyok
position startpos moves e2e4 e7e5
go depth 6
```

Supported commands:

| Command | Description |
|---|---|
| `uci` | Print engine identity and `uciok` |
| `isready` | Replies `readyok` |
| `ucinewgame` | Reset to the start position |
| `position startpos [moves ...]` | Set up the start position, optionally followed by moves |
| `position fen <fen> [moves ...]` | Set up a position from FEN |
| `go depth <n>` | Search to a fixed depth |
| `go movetime <ms>` | Search for a fixed time |
| `go wtime <ms> btime <ms> [winc <ms> binc <ms> movestogo <n>]` | Search with clock-based time management |
| `go infinite` | Search until stopped |
| `quit` | Exit |

## Debugging

Uncomment `#define DEBUG` in `defs.h` to enable the `ASSERT` checks used throughout the code.

`PerftTest(depth, pos)` in `perft.c` counts leaf nodes per root move. Compare its output against known perft numbers to validate move generation. `vice.c` includes a sample perft position (`PERFT`) and a tactical test position (`WAC1`) for this.

## Limitations

- Evaluation is material and piece-square tables only. There is no king safety, pawn structure, mobility, or endgame knowledge.
- No transposition table (only a PV table), no null-move pruning, and no late move reductions.
- No opening book or endgame tablebases.
- No UCI options (hash size and so on) are exposed.
- The time-handling code in `misc.c` is written mainly for Windows. The non-Windows branch has not been tested.

## Ideas for Improvement

- Transposition table with a configurable hash size
- Null-move pruning and late move reductions
- Richer evaluation: passed and isolated pawns, bishop pair, king safety, mobility, tapered endgame king tables
- Aspiration windows
- Opening book support

## Credits

Built following the structure of the Vice chess engine tutorial series by Bluefever Softwar
e.
