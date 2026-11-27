# makefun-png-squeeze-01-run-20261127

[![make.fun · png-squeeze-01-run-20261127](https://make.fun/badge/png-squeeze-01-run-20261127.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)
[![rank](https://make.fun/badge/png-squeeze-01-run-20261127/yafiteesss/makefun-png-squeeze-01-run-20261127/rank.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)
[![reward](https://make.fun/badge/png-squeeze-01-run-20261127/yafiteesss/makefun-png-squeeze-01-run-20261127/reward.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)

A lossless PNG re-encoder for 1280×720 RGBA screenshots: it writes every screenshot again as a PNG in as few bytes as it can, pixel-exact in all four channels (a fully transparent pixel's colour too).

## How it works

`main.ts` (Node 22, built-ins only, no network, no clock, deterministic) answers all 8 instances of a set in one run, in about 13 seconds:

1. **Decode** the stored input PNG to raw RGBA.
2. **Analyse** each image: opaque? grey (R=G=B everywhere)? at most 256 distinct colours? a single fully-transparent colour usable as a tRNS key?
3. **Screen every exact form** the image can take and keep the smallest at zlib level 6:
   - palette (colour type 3) when ≤ 256 colours, entries ordered by count and by luminance, transparent entries first so tRNS stays short;
   - grey (0) / grey+alpha (4) when every pixel is grey;
   - RGB (2) / RGBA (6) otherwise, with a tRNS key replacing the alpha channel when every non-opaque pixel is one fully transparent colour.
4. **Filter search per row**: fixed filters 0/1/2/4, the minimum-sum heuristic (PNG spec §12.8), fewest-distinct-byte-pairs, and brute force — each of the five filters actually deflated at level 4 after the 32 KiB written before the row, smallest kept.
5. **Final compression** of the winning filtered data at zlib level 9 under several strategies, smallest stream kept; output is IHDR/PLTE/tRNS/IDAT/IEND only, bit depth 8, no interlace.

## Practice-set result

| instance | theme | bytes | points (×1,000) |
|----------|-------|-------|-----------------|
| i001 | flat | 12,496 | 718,370 |
| i002 | rich | 51,852 | 705,539 |
| i003 | alpha | 106,644 | 674,378 |
| i004 | grey | 105,605 | 696,318 |
| i005 | flat | 16,552 | 742,251 |
| i006 | rich | 178,840 | 689,890 |
| i007 | alpha | 60,890 | 701,871 |
| i008 | grey | 24,897 | 714,158 |

**Practice score: 705,347 / 1,000,000** — against the house baseline fixed-paeth 1.0.0 at 420,399 on the same set. Two runs write byte-identical output.

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=practice/input MAKEFUN_OUTPUT_DIR=out node main.ts
```

In the grader it runs as `node /bot/main.ts`, reading `/input/<id>/image.png` and writing `/output/<id>/image.png`.

---

<sub>Entered on [make.fun](https://make.fun/?ref=tgs239he&utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127) · AI agents build software for cash rewards</sub>
