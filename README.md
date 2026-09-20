# Apex Legends pick rates by rank — daily open dataset

Daily pick share (% of active players whose selected legend is X) for all 28 Apex Legends legends,
sampled at 00:00 UTC, per rank board — Bronze, Silver, Gold, Platinum, Diamond, Master + Apex Predator,
the all-ranks board (since 2026-06-01) and the top-players ranked board (pick share of games, positive-RP
rate, season to date). Season 30 "Marked" and its patches are annotated.

Maintained by the [LFcarry](https://lfcarry.com) pro-analytics team. The same series drives the interactive
board on [lfcarry.com/guides/apex-legends-tier-list](https://lfcarry.com/guides/apex-legends-tier-list),
which turns it into tiers per rank band and explains the movement; this repository is the raw data behind it.

## Files

| file | what |
|---|---|
| `data/apex_pick_rates_long.csv` | long format: `date, board, legend, pick_pct` — 6,197 rows as of 2026-09-19 |
| `data/apex_tier_series.json` | the same series in the original wide format (`legends` column order → `boards.<board>.rows[date]`), plus `patches` (season/hotfix dates) and `fetchLog` (which boards were fetched each day) |
| `data/apex_tier_delta.json` | the 7-day movement contract: per legend and band, pick share, delta with a dead band + 2-day confirmation, editorial tier letter (source of truth for the published board) and an automatic pick-share quantile letter |
| `docs/summary.json` | boards, coverage dates, latest top-8 on the all-ranks board |

## Coverage (as of 2026-09-19)

| board | days | from | to |
|---|---|---|---|
| All ranks, any mode | 106 | 2026-06-01 | 2026-09-14 |
| Bronze · Silver · Gold · Platinum · Diamond · Master+Pred | 17 each | 2026-08-29 | 2026-09-14 |
| Top players ranked (pick share of games, positive-RP rate) | 11 | 2026-08-31 | 2026-09-14 |

Latest all-ranks top 8 (2026-09-14): Pathfinder 10.34%, Axle 8.72%, Loba 7.82%, Fuse 6.94%, Mad Maggie 6.75%,
Octane 6.15%, Wraith 5.10%, Bangalore 4.44%.

## How the numbers are made

- Source: [Apex Legends Status](https://apexlegendsstatus.com) public boards (`/lib/legendtrend.json` for the
  all-ranks trend, the per-rank pick-rate boards, and `/meta` for the top-players board), fetched once a day at
  00:00 UTC by a scheduled job. `fetchLog` records which boards succeeded on which day; a missing day means the
  fetch failed, not that the value was zero.
- `pick_pct` is the share of active players with that legend selected at the sample moment (all-ranks and rank
  boards) or the share of games on the top-players board. Rows sum to ~100 per board and day.
- Movement (`apex_tier_delta.json`) compares the last 7 days with the 7 before, ignores moves inside a dead band,
  and confirms a direction only when it holds for 2 consecutive days — this is what the arrows on the published
  board mean.
- Tiers (S/A/B/C) in `editorialTier` are set by the LFcarry analysts on top of the data; `autoTier` is a pure
  pick-share quantile, provided so you can compare the two.

## Use it

```python
import pandas as pd
df = pd.read_csv("data/apex_pick_rates_long.csv", parse_dates=["date"])
diamond = df[df.board == "Diamond"].pivot(index="date", columns="legend", values="pick_pct")
print(diamond.iloc[-1].sort_values(ascending=False).head(10))
```

Updates land daily. Issues and pull requests (new boards, corrections) are welcome.

## License and attribution

Derived series, code and documentation: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — credit
"LFcarry pro analytics, data via Apex Legends Status". Underlying figures are published by Apex Legends Status;
Apex Legends is a trademark of Electronic Arts / Respawn. This is an independent, unofficial dataset.
