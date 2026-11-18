# METHOD — how this forecast is made, and how honest it is

## What it forecasts

For each hour h = 2..24 of GTA VI's release day (2026-11-19 UTC), five quantiles
of the Twitch category's concurrent viewers in that hour — the number Twitch's
directory page shows for "Grand Theft Auto VI" (game id 1867279146). The program
(`main.ts`, Node 22, standard library only, no network) reads one case file
(`/input/<id>.in`: everything recorded up to the end of hour 1, hour 1's print
included) and writes `/output/<id>.out`: exactly 23 rows, each
`{"hour": h, "q": [q05, q25, q50, q75, q95]}`. The score is the mean pinball
loss on the log scale over every hour and quantile; lower is better.

A forecast here is **five numbers per hour, not one**. The median (q50) is the
best single guess. The 5%–95% band [q05, q95] is the honest part: an honest
90% band should contain the truth about 9 hours in 10. A forecast that reports
one number as if it were certain is not a forecast — it is a guess wearing a
suit. This one reports the range it actually believes.

## The method: cross-launch ratios, in log space

1. **Anchor on hour 1.** The case's hour-1 print `p1` is the only scale the
   program takes from the final case. Every forecast is `p1` times a ratio.
   GTA VI's hour 1 is already past every previous Twitch launch record
   (Streams Charts: 3.29M Twitch viewers on 18 Nov, before North America had
   unlocked), so any method that anchors on an absolute level from past
   launches would be wrong by construction. Ratios are scale-free.
2. **Learn the shape from past launches.** The case file carries 42 past big
   launches' hourly Twitch counts (SullyGnome hourly category averages, 14
   days before to 3 days after each release day). For each launch, divide every
   release-day hour by that launch's own hour-1 count. That gives the shape of
   a launch day as 23 ratios.
3. **Keep only the launches whose shape can transfer.** A launch enters the
   pool when (a) the house's own notes tier it "comparable", and (b) its
   hour-1 count is already at least 2% of its release-day peak — the data
   saying the launch was under way at 00:00 UTC. 19 of the 42 survive:
   ELDEN RING, Resident Evil Requiem, Cyberpunk 2077, Ghost of Yōtei, Monster
   Hunter Wilds, Call of Duty: Black Ops 6, Modern Warfare III, Black Ops 7,
   Modern Warfare 4, Saros, The Legend of Zelda: Tears of the Kingdom,
   Borderlands 4, Elden Ring Nightreign, Clair Obscur: Expedition 33, FINAL
   FANTASY VII REBIRTH, The Last of Us Part II, Ghost of Tsushima, Halo
   Infinite, Stellar Blade. The rule is written from the data, not a hand-picked
   list: a case file with different comparables gets its own pool by the same
   rule.
4. **Median ratio + cross-launch spread, in log space.** Per hour, the median
   of `ln(viewers_h / viewers_1)` across the pool is the log-median; the
   population standard deviation of those log-ratios is the spread, floored at
   0.12 and capped at 0.90 log units. The five quantiles are the log-normal
   quantiles at the scored levels (z = −1.645, −0.674, 0, 0.674, 1.645). Viewer
   counts are log-normal-ish, and the contest scores on the log scale, so the
   whole model lives in log space. The spread is wide exactly where launches
   disagree most — the evening peak — and narrow where they agree (the night
   dip). The pinball loss is a proper scoring rule for quantiles: it rewards
   reporting the quantiles you really believe, so the band is the belief, not
   decoration.

### The shape the pool shows (median ratio to hour 1, 19 launches)

| hour (UTC) | median | 10th pct | 90th pct |
|---|---|---|---|
| 02:00 | 0.87x | 0.74x | 1.23x |
| 03:00 | 0.75x | 0.47x | 1.08x |
| 04:00 | 0.73x | 0.46x | 0.90x |
| 05:00 | 0.80x | 0.61x | 1.15x |
| 06:00 | 0.95x | 0.51x | 1.29x |
| 07:00 | 0.98x | 0.58x | 1.27x |
| 08:00 | 0.94x | 0.51x | 1.96x |
| 09:00 | 0.92x | 0.48x | 2.19x |
| 10:00 | 0.94x | 0.43x | 2.21x |
| 11:00 | 0.93x | 0.47x | 2.14x |
| 12:00 | 0.98x | 0.47x | 2.14x |
| 13:00 | 1.12x | 0.51x | 2.59x |
| 14:00 | 1.30x | 0.57x | 3.17x |
| 15:00 | 1.52x | 0.65x | 4.38x |
| 16:00 | 1.65x | 0.69x | 5.05x |
| 17:00 | 1.77x | 0.76x | 4.80x |
| 18:00 | 1.95x | 0.91x | 5.42x |
| 19:00 | 2.40x | 0.95x | 5.66x |
| 20:00 | 2.37x | 0.90x | 8.67x |
| 21:00 | 2.33x | 0.92x | 8.71x |
| 22:00 | 2.30x | 0.87x | 6.99x |
| 23:00 | 2.18x | 0.78x | 6.13x |
| 24:00 | 2.04x | 0.72x | 4.79x |

The story the table tells matches the unlock clock: a night dip to ~0.73x at
04:00 UTC, a climb from 13:00 as the US wakes and plays, and a peak of ~2.4x
at 19:00–22:00 UTC (UK/Europe evening + US afternoon). GTA VI unlocks at
midnight local time, rolling — New Zealand first at 11:00 UTC on 18 Nov,
central Europe 23:00 UTC, the UK 00:00 UTC, the US east coast 05:00 UTC and
the west coast 08:00 UTC (Rockstar Support; US store listings use one
nationwide time, midnight Eastern). So hour 1 (00:00–01:00 UTC) already holds
the UK, Europe and Australasia, and the US arrives in hours 6–9 — the table's
rise is the unlocks arriving, not a guess.

### Worked example: Marvel's Wolverine (practice case p00)

Hour-1 print 23,357 viewers. The program's forecast (full output in
`examples/p00-forecast.json`):

| hour | q05 | q50 (median) | q95 |
|---|---|---|---|
| 02:00 | 14,513 | 20,353 | 28,545 |
| 12:00 | 7,619 | 22,843 | 68,489 |
| 19:00 | 16,429 | 56,094 | 191,530 |

Wolverine's real evening ran hotter than the median (its 19:00 print was
175,604, near this forecast's q95) — which is exactly why the band is wide:
one launch in five runs that hot. The score on that case still beat the house
baseline by a wide margin (see Validation).

## Assumptions, stated plainly

1. **Launch days rhyme.** A rolling midnight-local console launch in 2026
   follows the same hourly shape, in ratio terms, as the 19 comparable
   launches from 2020–2026. If GTA VI's audience behaves differently — for
   example because its hour 1 is already at its peak and there is no evening
   left to climb to — the median will over-forecast the evening. The band is
   the hedge against that.
2. **Hour 1 anchors everything.** If the final case's hour-1 print is small
   or missing, the forecast scales down with it; the program falls back to the
   last history point, then to 1 viewer, rather than crashing.
3. **SullyGnome hourly averages stand in for minute means.** A practice
   hour's print is one SullyGnome hourly average; the final's is the mean of
   ~60 minute reads. They are close, not identical.
4. **No streamer-level effects.** One giant streamer going live (or offline)
   can move an hour by hundreds of thousands of viewers. The model has no
   term for that — it is inside the spread, not predicted.
5. **No drops, no early access, no PC.** GTA VI launches with no Twitch Drops
   announced, no early access, and no PC version. Past launches with drops or
   early access (Hogwarts Legacy's 1.2M+ peak was helped by Drops; its
   early-access days ran three days before release) are excluded from the pool
   for exactly that reason.

## Data sources

- **Hourly Twitch counts:** SullyGnome.com hourly category averages, as
  carried in the case files (`comparables`, `history`). Source pages:
  https://sullygnome.com/game/Grand_Theft_auto_VI_1867279146 and the 41 other
  launch pages listed in the kit's `input/NOTICE.txt`. SullyGnome publishes no
  open licence; the figures are factual counts of Twitch's public directory,
  quoted with attribution. The practice answers were checked against
  SullyGnome's chart JSON for Marvel's Wolverine (verified alignment against
  the case file's own `history_hourly` tail).
- **GTA VI launch facts:** Rockstar Support — "midnight local time on
  November 19, 2026, in all regions", PS5 and Xbox Series X|S, no PC at
  launch; pre-load from 10 Nov, up to 212 GB.
- **Scale of the launch (context, not model input):** Streams Charts —
  GTA VI passed 3.29M Twitch viewers on 18 Nov, before North America
  unlocked, and 7.89M across platforms (preliminary); previous launch record
  was Valorant's closed beta at 1.74M (Apr 2020). TwitchTracker all-time peaks:
  Cyberpunk 2077 1,140,960; Grand Theft Auto V 1,140,747; ELDEN RING 910,225.
  Hogwarts Legacy ~1.2M+ (Stream Hatchet / ClutchPoints, helped by Drops).
- **Scoring:** pinball loss is the strictly consistent scoring function for
  quantiles — minimised by reporting the true quantile (see "Scoring rule",
  Wikipedia, and the interval-forecast evaluation literature, e.g. the
  COVID-19 Forecast Hub's weighted interval score, which generalises the
  pinball loss). The contest scores it on the log scale, so a 20% miss costs
  the same at 10,000 viewers as at 1,000,000.

## Validation

- **Leave-one-out over the 19-launch pool** (same code path, pinball loss on
  the log scale, mean over hours 2–24 and all five levels): **this model
  0.326, house baseline 0.403**. The baseline is flat persistence with a
  widening spread (the kit starter). The model beats it on the pool average
  and on 13 of the 19 launches individually.
- **Marvel's Wolverine, real practice answer** (SullyGnome hourly averages,
  2026-09-15): **this model 0.426, house baseline 0.744** — typical miss
  111.6% vs 188.8%, 90% coverage 0.78 vs 0.52.
- **House baseline on all four practice cases** (per the competition rules):
  about **0.387**. The model is built to beat that bar on the final; the
  practice is graded on past launches anyone can look up, so a practice score
  is no guarantee for the final — it is the best evidence available.
- **Determinism:** two runs on the four practice cases produce byte-identical
  outputs. The program is deterministic by construction (sorted inputs, no
  randomness, no clock).

## Limitations — what this forecast does not know

- It does not use the final case's `streams` (top 20 live streams) or
  `categories_hourly`: only the final case carries them, so no practice case
  can validate a rule built on them. A version that did would be guessing.
- It does not use GTA VI's own pre-release category history: sparse (sampled
  about every 12 hours) and pre-release hype is not launch-day shape.
- It cannot see Twitch Drops, a surprise streamer, a server outage, or a
  review-embargo lift — all of which move hours. Those live in the spread.
- If GTA VI's hour-1 print is already the day's peak (possible: the UK and
  Europe unlock before hour 1, and pre-release hype was already at record
  levels), the evening median will read high. The 5% quantile is the honest
  statement of that possibility: at 19:00 it sits near 1x hour 1, not 2.4x.
- The final's prints are minute means; the model's ratios come from hourly
  averages. The difference is small next to the cross-launch spread.
