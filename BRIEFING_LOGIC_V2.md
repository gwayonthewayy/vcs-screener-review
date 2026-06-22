# Briefing Selection Logic v2

## Fixed decision flow

The briefing selection order is:

1. Market regime and exposure multiplier
2. Independent weekly liquid-universe sector leadership
3. Company business directness to an actually leading sector
4. VCS/base quality and MA50 position
5. Latest announced earnings and OPM quality
6. Relative-strength line versus the market benchmark
7. Entry position, extension, liquidity and risk
8. Hard gate, then one weighted recommendation score

Daily watchlist candidates are never described as the whole market. Candidate files record the daily scan scope, while sector files record `leadership_universe_scope` separately. If the weekly universe is unavailable, the sector job marks its result `candidate_fallback_not_full_market`; it must not be interpreted as full-market leadership.

## Weekly universe refresh

Refresh the independent leadership universe and rolling daily watchlists manually:

```bash
.venv/bin/python scripts/refresh_weekly_universe.py --market both
```

Outputs are generated locally and ignored by Git:

```text
universes/kr_liquid_YYYYMMDD.csv
universes/kr_liquid_current.csv
universes/us_liquid_YYYYMMDD.csv
universes/us_liquid_current.csv
watchlists/kr_daily_current.txt
watchlists/us_daily_current.txt
```

The scanner automatically prefers a newer `*_daily_current.txt` over the legacy dated default path. This task does not register or modify cron.

The weekly refresh writes to temporary files first, validates minimum liquid-symbol count and coverage, and only then swaps the dated/current/watchlist files into place. A failed KR refresh never overwrites the US cache, and vice versa.

## Business directness

`business_directness` is one of `direct`, `adjacent`, `peripheral`, or `unrelated`. A company can be `direct` only when structured industry/business evidence matches a sector present in the current leadership result. Generic words such as `software`, `technology`, or `solutions` are not direct AI evidence. Ambiguous matches set `needs_sector_review=True` and remain observation-only.

The practical gate also requires `leadership_universe_scope == weekly_liquid_full_market_fresh` and `sector_leadership_score >= 60`. Stale cached context and candidate fallback stay in observation mode even if the business match itself looks strong.

## RS contract

- KR `.KS`: KOSPI
- KR `.KQ`: KOSDAQ
- US: QQQ, with SPY fallback
- Gate: RS data present and RS Line above EMA21
- Method: `relative_line_ema21_percentile_proxy`
- Confidence: `high`, `medium`, `low`, or `unavailable`

The candidate CSV stores `rs_score`, `rs_above_ma`, `rs_mom_5d`, `rs_mom_21d`, `rs_new_high`, `rs_new_high_before_price`, benchmark/source, method and confidence. Missing or below-EMA21 RS is observation-only.

## Earnings quality

The candidate CSV stores announced quarter, release date, source type and age in days. Quality is one of:

- `profitable_expansion`
- `profitable_slowdown`
- `turnaround_positive`
- `loss_narrowing`
- `distorted_or_missing`

Missing, stale or non-verifiable data fails closed. When both one-year and three-year OPM are negative, improvement is `loss_narrowing`, not profitable expansion.

## Recommendation gate and score

The hard gate runs before `recommend_score`. RS unknown/below EMA21, unverified earnings, ambiguous business classification, MA50 below/overextended position, high-risk biotech and distorted OPM are observation-only.

The 100-point score is:

| Component | Maximum |
| --- | ---: |
| Sector leadership | 15 |
| Business directness | 10 |
| Earnings quality | 20 |
| OPM quality | 15 |
| VCS/base quality | 15 |
| RS | 15 |
| Entry position | 5 |
| Liquidity/risk | 5 |

Sector plus directness is capped at 25 and earnings plus OPM at 35 by construction. VCS appears only once. Failed candidates retain `observation_score` and reasons, but `recommend_score` is blank.

## Output consistency

`canonical_practical_candidates()` is the only Top selector. Briefing Top, Telegram Top and main TradingView files use the same candidates and order. A missing `is_practical_recommendation` or `recommend_score` column is an error, not an implicit pass. Observation candidates are written only to observation sections and separate `TV_observation_*` files.

## Dry-run verification

Build summaries and TradingView files directly in a temporary directory. Do not invoke Telegram `sendMessage`/`sendDocument` or rclone during logic rehearsals. The production automation, cron entries and delivery credentials are outside this logic change.
