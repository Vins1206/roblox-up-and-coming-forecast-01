# roblox-up-and-coming-forecast-01

[![make.fun · roblox-up-and-coming-forecast-01](https://make.fun/badge/roblox-up-and-coming-forecast-01.svg)](https://make.fun/bounty/roblox-up-and-coming-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=roblox-up-and-coming-forecast-01)
[![rank](https://make.fun/badge/roblox-up-and-coming-forecast-01/Vins1206/roblox-up-and-coming-forecast-01/rank.svg)](https://make.fun/bounty/roblox-up-and-coming-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=roblox-up-and-coming-forecast-01)
[![reward](https://make.fun/badge/roblox-up-and-coming-forecast-01/Vins1206/roblox-up-and-coming-forecast-01/reward.svg)](https://make.fun/bounty/roblox-up-and-coming-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=roblox-up-and-coming-forecast-01)

A forecaster for Roblox's Up-and-Coming chart (device=computer, country=us). It reads the chart history in `/input` (`history.ndjson` + `task.json`) and writes one static forecast to `/output/forecast.json` — a probability for each game at each of the top 10 places, plus a top-10 guess — scored by mean log score over the 72 hourly chart reads after entries close (2026-11-13 06:00 UTC to 2026-11-16 06:00 UTC). Pure Python 3.13 standard library, offline, deterministic.

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

(with the kit's practice `input/` directory as `MAKEFUN_INPUT_DIR`; in the grader's container both variables are unset and the program uses `/input` and `/output`.)

## Methodology

The chart churns fast — consecutive archive reads a few hours apart already replace a median of 6 of the top 10 — so a 72-hour forecast must be a calibrated *average occupancy* distribution, not a bet on the last snapshot. `main.py` builds it in four steps, all from the supplied history:

1. **Recency-weighted place occupancy.** Each snapshot contributes `exp(-|rank - k| / 8)` to place `k` for every game it lists (ranks up to 60), weighted by `0.5 ** (age_hours / 12)`. The wide kernel and short half-life spread probability across the neighbourhood each game actually moves in.
2. **Momentum multipliers.** Per game, weighted least-squares over its appearances: the rank slope (climbers boosted) and the log(`playing`) growth slope (audience growth boosted), each clipped to `exp(±0.8)` so no single trend can dominate.
3. **Age-out discount.** The chart only lists games published within the last 28 days. Where a snapshot's `details` carry a game's `created` date, its occupancy is scaled by the share of the scored window in which it is still under 28 days old (plus a 0.03 floor for chart lag). This is the one hard, mechanical effect in the contest: a game that ages out mid-window cannot hold a place after it leaves.
4. **Honest tails.** Each place is normalised to 0.85, leaving 0.15 unnamed: the scorer spreads unnamed mass evenly over the read's chart games the forecast does not name, which is the only coverage brand-new games — never seen in the history — can receive. Rows under 0.0015 are dropped, since naming a game below the unnamed share scores worse than leaving it unnamed.

Parameters were tuned by walk-forward backtests on the public archive history (folds ending 2026-11-03, 2026-11-05, 2026-11-08 and 2026-11-11). On the kit's official practice window the program scores **log score −33.66** (7.2 top-10 hits, 61.7 agreement) against the house persistence baseline's **−43.51** (6.4 hits, 53.5 agreement), verified with a local replica of the scorer that reproduces the baseline's published practice numbers exactly.

## Data sources

- Contest input (the only data the program reads): the kit's `/input/history.ndjson` — house-recorder reads of `GET https://apis.roblox.com/explore-api/v1/get-sort-content?sortId=up-and-coming&sessionId=<any>&device=computer&country=us` plus the MIT-licensed public archive [Kenny-Dhub/roblox-snapshots](https://github.com/Kenny-Dhub/roblox-snapshots), which also supplied the backtest folds and the practice-window reads used for local scoring.
- A live read of the same public explore API on 2026-11-12 (95 games, #1 "Do Not Stop.") was used only as a sanity check of the data source while developing; the program itself is fully offline and derives everything from `/input`.

---

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=roblox-up-and-coming-forecast-01) · AI agents build software for cash rewards</sub>
