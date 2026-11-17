# GTA VI launch-day Twitch forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/usb-star001/gta6-forecast-01/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/usb-star001/gta6-forecast-01/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Forecasts how many people watch Grand Theft Auto VI live on Twitch in each hour
of its release day, 2026-11-19 (UTC), as five quantiles per hour (hours 2–24),
from everything recorded up to the end of hour 1. Built for the
[make.fun bounty](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
($250 total, scored by mean pinball loss on the log scale).

## The model

`main.ts` (Node 22, standard library only) fits a **comparable-launch ratio
curve at run time** from the case file's own `comparables` — 38 past big
launches in a practice case, 42 in the final:

1. Anchor on the case's hour-1 print `p1` (UK, central Europe and
   New Zealand/Australia are already unlocked in hour 1).
2. For each comparable launch, take log(`viewers_h / viewers_1`) per hour.
   The per-hour **weighted median** across launches is the day's shape: a dip
   to ~0.7× in the small hours, then a climb to a peak in hours 19–21 as the
   US unlocks (east coast 05:00 UTC, west coast 08:00 UTC).
3. Weight each launch by how close its own hour-1 size is to `p1`
   (`1 / (1 + |ln(v1/p1)|)`), and downweight launches whose hour 1 was still
   pre-unlock (peak/hour-1 > 20×) — their curves measure unlock timing, not
   the day's shape. Big launches (hour-1 ≥ 100k — ELDEN RING 875k, Cyberpunk
   2077 852k, Hogwarts Legacy 387k) peak at only ~1.25× hour 1, while
   small-hour-1 launches ramp 50–2000×; the weighting keeps the forecast in
   the right regime for whatever hour-1 print the final actually gives.
4. Spread is the weighted interquartile range of the log-ratios
   (σ = IQR / 1.349, clamped to [0.18, 0.85]); quantiles are lognormal around
   the median. The median curve is smoothed with a 3-hour moving average.

Full method, measured curves and validation: [MODEL.md](MODEL.md).
Grounding sources: [SOURCES.md](SOURCES.md).

## Validation (local, on kit data)

| Test | This model | House baseline |
| --- | --- | --- |
| Practice p00, Marvel's Wolverine (answer: SullyGnome hourly averages) | **0.464** | 0.744 |
| Leave-one-out, big launches only (hour-1 ≥ 100k) | **0.231** | 0.261 |
| Leave-one-out, all 38 comparables | **0.801** | 0.930 |

House baseline scores ~0.387 on the practice overall. 90% coverage on p00:
0.83 (baseline 0.52). Two runs of the program are byte-identical;
4 practice cases run in ~0.5 s total (limit: 8 s per case).

## Run it

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Reads every `*.in` in the input directory and writes one `<id>.out` per case:
`{"hours": [{"hour": 2, "q": [q05, q25, q50, q75, q95]}, …]}`.

## Layout

- `main.ts` — the forecaster (the entrypoint; no other language file at root)
- `MODEL.md` — method, measured curves, validation
- `SOURCES.md` — grounding sources for the numbers the model is built on
- `LICENSE` — MIT
