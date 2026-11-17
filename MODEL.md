# Model: comparable-launch ratio curve

## What is forecast

For each hour h = 2..24 of GTA VI's release day (2026-11-19 UTC), five
quantiles of Twitch concurrent viewers in the "Grand Theft Auto VI" category,
at levels 0.05 / 0.25 / 0.5 / 0.75 / 0.95, anchored on the case's hour-1 print
p1 (00:00–01:00 UTC). Scored by mean pinball loss on the log scale.

## Why ratios to hour 1

Hour 1 is already a launch audience: the UK unlocks at 00:00 UTC, central
Europe unlocked at 23:00 UTC the day before, and New Zealand/Australia have
been playing since 11:00 UTC Nov 18. So `viewers_h / viewers_1` is a shape,
not a level — and the level is whatever the final's hour-1 print turns out
to be, which the program reads from the case file at run time. Nothing about
GTA VI's absolute size is hardcoded.

## The measured shape (from the case file's comparables)

Median ratio to hour 1, all 38 comparables in practice case p00:

| hour | 2 | 3 | 4 | 5 | 6 | 8 | 10 | 12 | 15 | 19 | 24 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| median ratio | 0.87 | 0.75 | 0.73 | 0.75 | 0.79 | 0.90 | 0.91 | 0.98 | 1.52 | 2.40 | 2.04 |

Restricted to big launches (hour-1 ≥ 100k — the regime GTA VI belongs to):

| hour | 2 | 4 | 6 | 9 | 12 | 15 | 19 | 24 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| median ratio | 0.90 | 0.69 | 0.69 | 0.52 | 0.57 | 0.84 | 1.25 | 1.14 |

Two populations exist in the data. Launches whose hour 1 was still
pre-unlock (peak/hour-1 > 20×, e.g. Baldur's Gate 3 at 133×, Hollow Knight:
Silksong) ramp 50–2000× over the day; launches with a real hour-1 audience
peak at ~1.25×. The program tells them apart by weighting each comparable
by `1 / (1 + |ln(v1_comp / p1)|)` and downweighting pre-unlock hour-1s by
0.15 (×0.2 more above 100×). For a GTA VI hour-1 in the hundreds of
thousands, the big launches dominate the fit automatically; if hour 1
prints small, the rampers get more weight — the model adapts instead of
assuming a regime.

Individual big launches (ratio at hours 2 / 5 / 9 / 12 / 15 / 19 / 24):
ELDEN RING 0.92 / 0.66 / 0.53 / 0.57 / 0.76 / 0.95 / 0.72; Cyberpunk 2077
1.26 / 0.82 / 0.45 / 0.47 / 0.76 / 0.94 / 0.55; Hogwarts Legacy 0.89 / 0.51 /
0.43 / 0.51 / 0.90 / 1.26 / 1.11; Diablo IV 1.00 / 0.83 / 0.86 / 0.99 / 1.06 /
1.25 / 0.96; Zelda: Tears of the Kingdom 0.83 / 1.08 / 1.03 / 1.49 / 2.24 /
2.66 / 1.90. The dip in the small hours and the evening peak are common;
the peak's height is what varies — hence a wide spread late in the day.

## Spread

Per hour, σ = (weighted IQR of log-ratios) / 1.349, clamped to [0.18, 0.85].
The IQR of log-ratios is ~0.25 at hour 2 and ~2.1 at hour 19, so the 90%
band is narrow early and wide late: at hour 19 the q05–q95 band spans
roughly 0.25×–4× the median. Quantiles are lognormal:
`q_τ = p1 · exp(med_h + z_τ · σ_h)`. The median log-ratio curve is smoothed
with a 3-hour moving average. With no usable comparable for an hour, the
program falls back to the house baseline's spread (σ = 0.15·√(h−1)) around p1.

## Validation (local)

Scoring is mean pinball loss over the 23 hours × 5 quantiles, exactly as the
kit scores it. The practice cases' answers are not in the kit files, so:

- **Practice p00, Marvel's Wolverine** — answer taken from SullyGnome's
  published hourly averages for 2026-09-15 (hour-1 print 23,357; peak
  212,243 at hour 21, 9.1× hour 1):
  this model **0.4637** vs house baseline 0.7437; typical miss 122% vs
  189%; 90% coverage 0.83 vs 0.52. The model still under-predicts
  Wolverine's late ramp (median ratio 2.14× vs 7.5× actual) — Wolverine is
  PS5-only with a small hour 1 — but the wide late-day spread keeps the
  loss well below baseline.
- **Leave-one-out over the 38 comparables** (each launch predicted from the
  other 37): model **0.8005** vs baseline 0.9296.
- **Leave-one-out over big launches only** (hour-1 ≥ 100k): model **0.2306**
  vs baseline 0.2611.
- **Variants tested and rejected**: no pre-unlock downweighting (p00 0.403,
  but LOO-big 0.244 — worse where it matters); weighted mean of log-ratios
  instead of median (p00 0.246, but LOO-big 0.321 — worse than baseline in
  GTA VI's regime); blending median with 25–50% mean (p00 0.449–0.457,
  LOO-big 0.232–0.235 — no gain). The shipped model is the plain weighted
  median, variant A.

The house baseline scores ~0.387 on the practice overall (kit RULES.txt).
This model beats it on every local test; the remaining risk is the house
model forecast, whose loss is unknown.

## What the program does not use

No hardcoded GTA VI numbers, no network, no clock, no randomness. The
case's `history`, `streams` and `categories_hourly` are available but not
needed: the comparables already carry the day's shape, and hour-1 channels
showed no additional signal in the comparables. The program reads only the
case file it is given and is deterministic — two runs are byte-identical.
