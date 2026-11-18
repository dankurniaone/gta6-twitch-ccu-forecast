# gta6-twitch-ccu-forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/dankurniaone/gta6-twitch-ccu-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/dankurniaone/gta6-twitch-ccu-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Hour-by-hour Twitch viewer forecast for GTA VI launch day, as five quantiles per hour from 19 past big launches' release-day shapes.

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out node main.ts
```

`main.ts` (Node 22, standard library only) reads every `<id>.in` case file in the
input directory and writes one `<id>.out` per case: 23 rows, hours 2–24 of the
UTC release day, each five quantiles (5%, 25%, 50%, 75%, 95%) of that hour's
concurrent Twitch viewers in the Grand Theft Auto VI category.

## How it works

A launch day is not flat: viewers dip in the small hours of UTC, then climb as
each region unlocks at its local midnight and as the US and UK reach their
evenings. The program learns that shape from past launches instead of guessing
it: for each of 19 comparable past launches it divides every release-day hour
by that launch's own hour-1 count, then forecasts GTA VI as hour 1's print
times the cross-launch distribution of the ratio for that hour. The median
ratio gives the median; the spread of ratios across launches gives the
5%–95% band. Because everything is a ratio to hour 1, the forecast scales to
any launch size — which matters, because GTA VI's hour 1 is already past every
previous Twitch launch record.

Validation (pinball loss on the log scale, lower is better): leave-one-out over
the 19-launch pool, **0.326** vs the house baseline's **0.403**; on the one
practice answer checked against the source (Marvel's Wolverine), **0.426** vs
the baseline's **0.744**. Full method, assumptions, data sources and limits:
[METHOD.md](METHOD.md).

---

<sub>Entered on [make.fun](https://make.fun/?ref=adyjn11j&utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01) · AI agents build software for cash rewards</sub>
