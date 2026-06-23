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

The weekly refresh requires explicit source provenance before liquidity processing. KR requires successful KOSPI and KOSDAQ sources. US requires successful NASDAQ, NYSE, and NYSE American sources. A missing or failed required source rejects the entire market refresh; symbol-count heuristics cannot convert a partial fetch into a valid snapshot. Snapshot metadata is validated row by row. `market`, `universe_scope`, `universe_refreshed_at`, `universe_symbol_count`, `liquid_symbol_count`, `liquidity_coverage_pct`, `universe_required_sources`, `universe_source_status`, `universe_required_source_count`, and `universe_successful_source_count` must be non-null, stripped, and identical across every row. `symbol` must be non-null, non-empty, stripped, and unique. Partial rows, blank metadata, or mixed metadata invalidate the snapshot and force `candidate_fallback_not_full_market`. The analysis-ready liquidity denominator uses `ANALYSIS_READY_LIQUIDITY_RATIO=0.5`; a regression now pins exact below/at/above-half-threshold behavior so denominator tuning cannot silently change coverage.

Fresh cache validation is shared by KR and US and requires all of the following:

- non-empty CSV and complete snapshot schema
- every required metadata field present on every row
- `universe_scope == weekly_liquid_full_market`
- one parseable, non-future `universe_refreshed_at`
- timestamp within the configured freshness window
- internally consistent universe/liquid counts and coverage percentage
- complete required-source list and successful source-status metadata

Missing, malformed, future, expired, blank, or internally inconsistent metadata is never labeled fresh. Valid snapshots older than the fresh window may be used only as `stale_cached_market_context`; malformed, partial, or expired snapshots fall back to candidate context and cannot produce practical recommendations.

Publishing is transactional at the snapshot-set level. Dated CSV, current CSV, and daily watchlist are staged and validated before replacement. Publish order is dated, watchlist, then current; current is the final commit marker. Any intermediate failure restores the prior current/watchlist generation and cleans temporary files. A failed KR refresh never overwrites the US cache, and vice versa.

## Business directness

`business_directness` is one of `direct`, `adjacent`, `peripheral`, or `unrelated`. A company can be `direct` only when structured `industry`/business tags clearly match a current leader, or when the business description itself describes the company’s own product, technology, or service in that sector. Customer-only phrasing such as `serves semiconductor manufacturers`, `customers include semiconductor companies`, `logistics for chip makers`, `insurance products for data-center operators`, or `consulting to semiconductor companies` does not qualify as direct. `theme_hint`, generic sector names, and derived theme labels are auxiliary evidence only and cannot independently produce `direct`. Generic words such as `software`, `technology`, or `solutions` are not direct AI evidence. Ambiguous and auxiliary-only matches set `needs_sector_review=True` and remain observation-only.

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

Missing, stale, future-dated or non-verifiable data fails closed. The quarter must parse, must not end after the release date, and must be plausibly linked to that release. The release source must be one of the explicit allowlist values: `official`, `provisional`, `preliminary`, `dart_filing_date`, `dart_provisional_fair_disclosure`, `sec_filing_date`, `공식`, or `잠정`. Release timestamps are compared to `as_of` without day-level truncation; a later timestamp on the same date is still future. A future release retains its date and negative age for diagnostics but is `distorted_or_missing` with `earnings_verified=False`. Any source outside the exact allowlist, including substring lookalikes such as `unofficial`, `secondary estimate`, or `unofficial preliminary data`, fails closed. When both one-year and three-year OPM are negative, improvement is `loss_narrowing`, not profitable expansion.

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

The frozen positive fixture verifies three practical symbols in identical Markdown, Telegram, and TradingView order, disjoint observation output, and no observation backfill when practical count is zero. A builder-level fixture also exercises `scripts/build_market_briefing.py` against real temporary files to confirm the same selector order reaches Markdown, Telegram Top, and TradingView main outputs. The current regression set reaches the real file I/O path and the full suite contains 65 tests.
