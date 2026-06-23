# Dry Run Evidence

## Scope

The final remediation verification does not invoke Telegram or Google Drive delivery. Cron, `.env`, and delivery configuration remain unchanged.

## Production Entry-Point Call Flow

`scripts/run_daily_briefing.sh` remains the live entry point:

- `--market kr`: KR `full_market_scan.py` -> KR leadership jobs -> `scripts/build_market_briefing.py --market kr` -> TradingView KR -> Telegram summary builder.
- `--market us`: US `full_market_scan.py` -> `us_sector_leadership.py` -> `scripts/build_market_briefing.py --market us` -> TradingView US -> Telegram summary builder.
- `--market both`: both scans and leadership jobs -> both market briefings -> integrated briefing -> KR/US TradingView -> integrated Telegram summary builder.

The briefing builders call `candidate_enrichment_v2.prepare_candidates()`, which calls `briefing_logic_v2.enrich_recommendations()`. Markdown, Telegram Top, and TradingView main all select through `canonical_practical_candidates()`.

## Weekly Universe State

The live isolated refresh produced fresh weekly-universe files in temporary output only. The operational cache remained unchanged.

One isolated live refresh and follow-up dry-run were made:

```bash
.venv/bin/python scripts/refresh_weekly_universe.py --market both --output-dir /tmp/vcs-live-XXXXXX/universes --watchlist-dir /tmp/vcs-live-XXXXXX/watchlists
```

The isolated run completed with KR coverage `79.2%` and US coverage `91.9%`. Fresh weekly snapshot files were generated in the temporary directory, the follow-up dry-run consumed that fresh snapshot, and the operational cache stayed unchanged. Telegram and Google Drive were not invoked.

## Frozen Positive E2E Fixture

The deterministic `test_fresh_positive_fixture_has_identical_markdown_telegram_and_tv_order` fixture uses fresh leadership scope, verified earnings, valid RS, direct semiconductor business evidence, and passing chart/risk fields.

Expected and observed practical order:

```text
MTSI
SMTC
LSCC
```

| Consumer | Symbols/order |
| --- | --- |
| Markdown practical Top | `MTSI`, `SMTC`, `LSCC` |
| Telegram Top | `MTSI`, `SMTC`, `LSCC` |
| TradingView main upload | `MTSI`, `SMTC`, `LSCC` |

## Builder-Level Positive Fixture

The `test_builder_level_fresh_positive_fixture_keeps_order_and_disjoint_outputs` fixture goes through the real builder path with temporary on-disk files.

Observed properties:

- fresh leadership scope is preserved
- verified earnings and direct business evidence survive through the builder
- the same practical symbol order reaches Markdown, Telegram Top, and TradingView main output
- practical and observation TradingView sets remain disjoint
- builder output does not backfill observations when the practical set is empty

This gives file-I/O evidence for the canonical selector path without invoking Telegram or Google Drive delivery.

Observation fixtures include high-risk biotech and below-EMA21 RS cases. The TradingView practical/observation intersection is empty. The separate zero-practical fixture confirms Telegram emits only the no-practical message and does not promote observations.

## Negative E2E Boundaries

- Missing/malformed/future timestamp: fallback, never fresh.
- Valid but stale timestamp: stale context only; practical gate remains closed.
- Wrong scope, empty CSV, invalid counts/coverage, or source metadata: fallback.
- KOSPI success with KOSDAQ failure: market refresh fails and old generation is preserved.
- US missing a required core exchange source: market refresh fails and old generation is preserved.
- Publish failure between watchlist and current: rollback restores the old generation.
- Future earnings release: unverified `distorted_or_missing`.
- Theme-only semiconductor hint: review-only, not direct.
- Customer-only business phrasing: review-only, not direct.

## Remaining Production Risk

The live positive path is now verified. Remaining risk is operational rather than functional: the `ANALYSIS_READY_LIQUIDITY_RATIO=0.5` heuristic, Yahoo provider fragility, and the daily leadership label / product wording follow-up still need periodic review.
