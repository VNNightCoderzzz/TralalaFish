# TralalaFish

**TralalaFish** is a UCI chess engine written in C++20 for Windows x64.
This release packages the engine and its official neural network (NNUE) into a
**single `.exe` file** - download it, drop it anywhere, load it into your GUI, and play.

> **Note:** this repository ships the **binary release only**. The source code is
> not published, so this README contains no build instructions.

- Engine version: `TralalaFish 1.0.3`
- Evaluation: NNUE HalfKA with 4 king buckets, plus an HCE fallback
- Multi-threaded, configurable hash, ready for head-to-head / SPRT testing

---

## Highlights

- **One file.** `TralalaFish.exe` carries the official NNUE network inside it.
  No separate `.nnue` file to ship, no paths to configure.
- **Automatic HCE fallback.** If no usable network is available, the engine prints
  a clear message and falls back to HCE evaluation instead of stopping.
- **Stronger than HCE and Sunfish.** The final net beats HCE by **+40 Elo** and
  Sunfish by **+626 Elo** (see [measured strength](#measured-strength)).

---

## Download

Get `TralalaFish.exe` from this repository's **[Releases](../../releases)** page.

| Artifact | Size | Notes |
|---|---|---|
| `TralalaFish.exe` | `49,389,007` bytes | Engine + final NNUE embedded |

Integrity check (SHA-256 of the network embedded in the executable):
`28bb63c4d933ab9de00757abb4f3a4a9d11bc3e391dfca8bf13cc9a993d65efb`

---

## System requirements

- Windows 10/11 **x64**.
- Nothing else to install. The engine is a console program that talks over
  **stdin/stdout** using the UCI protocol.

---

## Usage

### With a chess GUI

The engine follows the UCI standard, so it works with any popular GUI (Arena,
Cute Chess, BanksiaGUI, Nibbler, en-croissant, ...):

1. In the GUI, choose **Add / New Engine** and point it to `TralalaFish.exe`.
2. Set **Threads** and **Hash** to match your machine.
3. Enable **NNUE** and start playing.

### From the command line

```text
uci
isready
position startpos
go movetime 1000
```

The engine replies with `bestmove ...`, like any other UCI engine.

---

## UCI options

| Option | Type | Default | Meaning |
|---|---|---|---|
| `Hash` | spin (1-32768) | `256` | Transposition-table size in MB |
| `Threads` | spin (1-256) | `1` | Number of search threads |
| `Clear Hash` | button | - | Clears the transposition table |
| `NNUE` | check | `false` | Enable/disable NNUE evaluation |
| `NNUEPost` | check | `false` | Verbose NNUE logging |
| `EvalNoise` | spin (0-500) | `0` | Adds noise to the evaluation (for self-play testing) |

The HCE tuning parameters are also advertised over `uci` and can be changed with
`setoption`.

---

## Measured strength

Direct head-to-head matches, `100 ms` per move, 4 games in parallel, `Hash 64`,
`Threads 8`. **A** = TralalaFish with the final net.

| Opponent | Games | A W / D / L | Score | Elo (A-B) | 95% CI |
|---|---:|---:|---:|---:|---:|
| Sunfish | 113 | 109 / 2 / 2 | 0.973 | **+626** | [+499, +1200] |
| HCE (1.0.3, no NNUE) | 1000 | 425 / 264 / 311 | 0.557 | **+40** | [+21, +58] |
| Previous b4 net | 1000 | 318 / 419 / 263 | 0.527 | **+19** | [+3, +36] |

-> The final net is **clearly stronger** than Sunfish, than HCE, and than the
previous b4 network.

---

## The NNUE network (`halfka14m_b4_c256_final`)

| Property | Value |
|---|---|
| Architecture | HalfKA, `45056 -> 256 -> 1`, **4** king buckets |
| File format | TralalaFish NNUE VERSION 2 |
| Labels | static evaluation, ~12.93M positions |
| Calibrated scale | `179` |
| File size | `46,142,504` bytes |
| MD5 | `d3a736c6be7eb32c53eacba82221218b` |
| SHA-256 | `28bb63c4d933ab9de00757abb4f3a4a9d11bc3e391dfca8bf13cc9a993d65efb` |

The network ships embedded inside `TralalaFish.exe`. You can extract it to a
standalone `.nnue` file for inspection or reuse:

```text
nnue-embed-dump halfka14m_b4_c256_final.nnue
```

### Verification performed

- **Integration:** 60,782 positions - `0` feature differences and `0 cp`
  evaluation differences versus the reference implementation.
- **Embedding:** maximum **`0 cp`** difference over 8,226 positions - the embedded
  data matches the original network byte for byte.
- **Single file:** a directory containing only `TralalaFish.exe` still loads NNUE
  and searches normally, with no error messages.

---

## Command-line tools (optional)

Besides the standard UCI loop, the engine exposes a few helper commands for
testing:

| Command | Purpose |
|---|---|
| `eval` | Prints the static evaluation of the current position (`static eval: N`, or `-2` when using HCE) |
| `hce-eval` | Prints the pure-HCE evaluation |
| `nnue-embed-dump <path>` | Writes the embedded network to a `.nnue` file |
| `nnue-dump <games> <plies> <seed> [net]` | Dumps features/scores per position (for validation) |

---

## Credits

- TralalaFish is developed by the **TralalaFish contributors**.
- The engine draws ideas and techniques from the wider open-source chess
  community.
- **Sunfish** was used as a test opponent.

See the `LICENSE` file on the release page (if present) for the terms of use.

---

## Contact

Open an **Issue** in this repository if you find a bug or want to share feedback.
