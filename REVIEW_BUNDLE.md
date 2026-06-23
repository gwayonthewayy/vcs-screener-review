# VCS Screener V2 Review Bundle

Date: 2026-06-23

## Review Context

This bundle is for final external review of `fix/briefing-logic-v2`.

### Background

The intended selection flow is:

market regime
-> independent whole-market sector leadership
-> direct business relationship to the leading sector
-> VCS 13/63/50 and MA50 position
-> latest reported earnings, YoY, rolling 4Q/12Q OPM
-> market-relative RS line / EMA21
-> chase risk, liquidity, and entry location
-> hard gates first, then recommendation ranking

Key operating rules:

- VCS threshold is 45.
- ATRP minimum is 2.
- Candidates may stay in the scan universe up to 10% below MA50.
- RS unknown or below EMA21 is observation only.
- Missing or stale earnings release date / quarter / source is observation only.
- Biotech, OPM distortion, loss-heavy names, and ambiguous business mapping stay observational.
- Practical recommendations are not backfilled from observation names.

## Branch and Base

- Branch: `fix/briefing-logic-v2`
- Base commit: `107103b7520437ab9bacd4c76ed669feaa93f613`
- Prior remediation tip: `1758fde0caca0434051ccc452fd1c926657d652a`
- Final feature tip: `4c27f2e38e1261ffc70f27637b26930d1205e8b0`

## Scope

The branch keeps cron, Telegram, Google Drive, and `.env` unchanged.
The work focuses on the briefing selection pipeline, weekly universe safety, and review evidence.

## Changed Files and Roles

| File | Role |
| --- | --- |
| `briefing_logic_v2.py` | Canonical hard-gate policy, score composition, exact earnings-source allowlist, business directness, and fail-closed recommendation selection. |
| `candidate_enrichment_v2.py` | Adds earnings, RS, and leadership-scope enrichment fields consumed by the canonical selector. |
| `scripts/refresh_weekly_universe.py` | Safely refreshes weekly liquid universes and daily watchlists without overwriting good cache on failure. |
| `scripts/run_daily_briefing.sh` | Orchestrates KR/US/both execution paths and keeps market-specific file names, Telegram, and upload packaging aligned. |
| `scripts/build_market_briefing.py` | Builds one-market daily markdown with explicit scope labels and practical-only Top sections. |
| `build_whole_market_briefings.py` | Builds split KR/US whole-market briefings from the same canonical selector. |
| `build_deep_minervini_briefing.py` | Builds the integrated briefing from the same canonical practical selector. |
| `scripts/build_tradingview_watchlists.py` | Emits practical TV files and separate observation-only TV files. |
| `scripts/build_telegram_summary.py` | Emits Telegram summaries from the same canonical practical selector. |
| `tests/test_briefing_logic_v2.py` | Unit and pipeline-contract coverage for the V2 selection contract. |
| `tests/test_weekly_universe_refresh.py` | Cache-preservation tests for weekly refresh failures and low-coverage cases. |
| `weekly_universe_cache.py` | Shared strict weekly snapshot schema, freshness, row-level metadata consistency, count/coverage, and source-provenance validation. |
| `tests/test_weekly_universe_cache.py` | Missing/malformed/future/stale metadata and KR title regression coverage. |
| `docs/briefing_logic_v2.md` | Selection-policy documentation and operational notes. |
| `docs/automation.md` | Run-mode and cron documentation, including the VCS threshold wording fix. |
| `install.cmd` | Deleted Windows-specific artifact that is not part of the Linux server automation path. |

## Final Hard Gate Order

The canonical gate in `briefing_logic_v2.enrich_recommendations()` is:

1. Leadership scope must be `weekly_liquid_full_market_fresh`.
2. `sector_leadership_score` must be present and at least 60.
3. `business_directness` must be `direct` or `adjacent`, and `needs_sector_review` must be false.
4. Earnings must be verified and one of `profitable_expansion`, `profitable_slowdown`, or `turnaround_positive`.
5. RS must be present and `rs_above_ma` must be true.
6. MA50 gap must not be below -10%; if below MA50 but within -10%, the name stays in observation as `MA50 회복 대기`.
7. MA50 extension above the allowed range, extended bases, volatile bases, and weak VCS labels fail the gate.
8. ATRP above the policy cap fails the gate.
9. Biotech and OPM-distorted / loss-heavy names fail the gate.

## Recommendation Score

`recommend_score_method = v2_gate_then_weighted_100`

Component weights:

- sector leadership: 15
- business directness: 10
- earnings quality: 20
- OPM quality: 15
- VCS / base quality: 15
- RS: 15
- entry position: 5
- liquidity / risk: 5

Notes:

- The hard gate runs before scoring.
- Sector + directness effectively caps at 25.
- Earnings + OPM effectively caps at 35.
- VCS is not double-counted.
- Practical names keep `recommend_score`; observation names keep `recommend_score = NaN` and use `observation_score`.

## Weekly Universe Policy

Weekly leadership is independent from the daily scan universe.

- Fresh: `leadership_universe_scope == weekly_liquid_full_market_fresh`
- Stale: `stale_cached_market_context`
- Fallback: `candidate_fallback_not_full_market`

The practical gate only accepts fresh leadership scope.
Fallback leadership is allowed for observation and diagnostics but must not be treated as full-market leadership.

The weekly refresh requires explicit source provenance: KOSPI and KOSDAQ for KR; NASDAQ, NYSE, and NYSE American for US. Any missing/failed required source rejects the market refresh before liquidity processing.

The weekly refresh stages and validates the dated CSV, current CSV, and daily watchlist as one generation. It publishes current last as the commit marker and rolls back current/watchlist/dated files on intermediate failure. If refresh fails, the existing generation is preserved.

## Daily Watchlist vs Whole-Market Leadership

- Daily watchlist candidates are the scan universe for the daily briefing.
- Whole-market leadership is the independent weekly liquid universe used to judge sector strength.
- The daily scan should never be described as the whole market.
- If the weekly universe is missing, the leadership job must say fallback explicitly rather than pretending to be whole-market leadership.

## Data Flow

### Earnings

`full_market_scan.py` computes and carries:

- `earnings_quarter`
- `earnings_release_date`
- `earnings_source_type`
- `earnings_age_days`
- `earnings_good`
- `announced_revenue_current`
- `opm_latest_q`
- `opm_roll4q`
- `opm_roll12q`

`candidate_enrichment_v2.py` reads those fields and fail-closes if they are missing or stale.

### RS

RS fields are written into candidate CSVs and cache files.

- KR benchmark mapping: `.KS -> KOSPI`, `.KQ -> KOSDAQ`
- US benchmark mapping: `QQQ` with `SPY` fallback

Stored fields include:

- `rs_benchmark`
- `rs_benchmark_source`
- `rs_score`
- `rs_above_ma`
- `rs_mom_5d`
- `rs_mom_21d`
- `rs_new_high`
- `rs_new_high_before_price`
- `rs_method`
- `rs_confidence`
- `rs_line`
- `rs_ma`
- `rs_ma_gap_pct`
- `rs_asof`

If RS cannot be computed, the candidate remains observation only.

## Automation Call Flow

`scripts/run_daily_briefing.sh` is the live orchestration entrypoint.

Shared flow:

1. `git pull --ff-only`
2. Activate `.venv`
3. Validate Python dependencies
4. Run KR and/or US scan
5. Add detailed theme enrichment
6. Build KR/US sector leadership
7. Build the relevant briefing markdown and DOCX
8. Build TradingView watchlists
9. Build Telegram summary
10. Send Telegram message and document attachments
11. Upload optional Google Drive package with `rclone`

Market-specific behavior:

- `--market us`: US scan, US briefing, US TV, US Telegram package
- `--market kr`: KR scan, KR briefing, KR TV, KR Telegram package
- `--market both`: KR + US scan, split market briefings, integrated briefing, integrated Telegram package, KR/US TV, optional combined Google Drive upload

## Resolved Issues

- Daily candidate universe is no longer described as the whole market.
- Practical and observation candidates are separated in Markdown, Telegram, and TV outputs.
- QTTB / RLYB style biotech names are observation-only.
- RS unknown or below EMA21 is practical-fail closed.
- The weekly refresh no longer overwrites a good cache on DNS / fetch failures or low-coverage refreshes.
- Missing, malformed, future, wrong-scope, count-inconsistent, partially null, or source-incomplete weekly metadata can no longer be labeled fresh.
- Partial KOSPI/KOSDAQ or US core-exchange refreshes cannot replace the current cache.
- Snapshot publish failure between watchlist and current replacement rolls back the whole generation.
- Future earnings releases fail closed, exact source-type allowlisting rejects substring lookalikes, and customer-only theme labels cannot independently create direct business classification.
- KR fresh scope now renders the full-market title using the canonical fresh constant.
- The VCS threshold wording is now consistent at 45.
- The analysis-ready coverage denominator is pinned by `ANALYSIS_READY_LIQUIDITY_RATIO=0.5`, and the boundary regression now covers below/at/above-half-threshold behavior.
- Windows-only `install.cmd` is removed from the Linux server review path.

## Residual Risks

- `ANALYSIS_READY_LIQUIDITY_RATIO=0.5` remains a heuristic, but it is now pinned by a boundary regression.
- Yahoo provider fragility remains a live dependency.
- Daily leadership label/product wording follow-up remains.

## Main Merge Checklist

Before any merge to `main`:

1. Verify fresh weekly universe creation on live network once.
2. Re-run `scripts/run_daily_briefing.sh --market us`, `kr`, and `both`.
3. Confirm Telegram and Drive attachments still match the canonical selector.
4. Confirm no stale/fallback leadership leaks into practical Top lists.
5. Confirm no uncommitted changes remain.
6. Confirm `review.patch` is up to date.
