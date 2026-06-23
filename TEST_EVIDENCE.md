# Test Evidence

## Final Review Remediation Run

Commands:

```bash
.venv/bin/python -m unittest discover -s tests -v
bash -n scripts/run_daily_briefing.sh
.venv/bin/python -m py_compile briefing_logic_v2.py candidate_enrichment_v2.py weekly_universe_cache.py scripts/refresh_weekly_universe.py full_market_scan.py kr_market_sector_leadership.py us_sector_leadership.py scripts/build_market_briefing.py scripts/build_telegram_summary.py scripts/build_tradingview_watchlists.py
git diff --check origin/main...HEAD
```

Current result: 64 tests passed. Bash syntax, Python compilation, and diff whitespace checks passed. Final verification is rerun immediately before commit.

## Test Inventory

### Business and earnings

- `test_ambiguous_match_requires_review`
- `test_ebay_and_dbd_are_not_ai_direct_from_software_word`
- `test_mtsi_is_direct_when_semiconductors_are_actually_leading`
- `test_theme_hint_semiconductor_alone_is_not_direct`
- `test_semiconductor_customer_mention_is_not_direct`
- `test_data_center_customer_mention_is_not_ai_direct`
- `test_logistics_for_chipmakers_requires_review`
- `test_us_common_stock_name_filters_non_common_listings`
- `test_kr_common_stock_name_filters_special_listings`
- `test_future_release_date_fails_closed`
- `test_same_day_future_release_time_fails_closed`
- `test_quarter_after_release_date_fails_closed`
- `test_missing_or_stale_earnings_fail_closed`
- `test_unofficial_earnings_source_fails_closed`
- `test_secondary_estimate_source_fails_closed`
- `test_exact_official_and_provisional_sources_pass`
- `test_two_negative_opm_periods_are_loss_narrowing`

### Recommendation, market, and output contracts

- `test_missing_market_data_is_neutral_not_bearish`
- `test_biotech_is_observation_only`
- `test_fresh_scope_and_sector_score_gate_practical`
- `test_ma50_minus_ten_percent_is_retained_but_not_immediate`
- `test_missing_practical_column_fails_closed`
- `test_practical_top_does_not_backfill_observations`
- `test_rs_unknown_or_below_ema21_is_not_practical`
- `test_sector_leadership_changes_recommend_score`
- `test_custom_watchlist_is_not_overridden`
- `test_rolling_watchlist_replaces_stale_dated_default`
- `test_rs_benchmark_mapping`
- `test_tv_export_fails_when_practical_schema_is_missing`
- `test_verified_direct_candidate_can_pass_end_to_end`
- `test_main_and_observation_tv_files_are_disjoint`
- `test_fresh_positive_fixture_has_identical_markdown_telegram_and_tv_order`
- `test_builder_level_fresh_positive_fixture_keeps_order_and_disjoint_outputs`
- `test_zero_practical_candidates_do_not_backfill_telegram_top`

### Weekly cache validation

- `test_missing_timestamp_column_is_not_fresh`
- `test_unparseable_timestamp_is_not_fresh`
- `test_future_timestamp_is_not_fresh`
- `test_wrong_universe_scope_is_not_fresh`
- `test_partial_null_market_row_is_not_fresh`
- `test_partial_blank_scope_row_is_not_fresh`
- `test_partial_null_snapshot_metadata_is_not_fresh`
- `test_inconsistent_snapshot_metadata_across_rows_is_not_fresh`
- `test_empty_weekly_csv_is_not_fresh`
- `test_invalid_count_or_source_metadata_is_not_fresh`
- `test_invalid_coverage_metadata_is_not_fresh`
- `test_failed_source_status_metadata_is_not_fresh`
- `test_valid_fresh_snapshot_is_accepted`
- `test_valid_old_snapshot_is_stale_not_fresh`
- `test_kr_fresh_scope_markdown_title_is_full_market`

### Refresh provenance and transaction safety

- `test_invalid_crumb_retry_uses_fresh_session`
- `test_unknown_failure_is_classified_as_unknown_error`
- `test_liquidity_failure_classifier_maps_common_yfinance_messages`
- `test_liquidity_rows_retries_invalid_crumb_with_fresh_session`
- `test_failed_symbol_reason_aggregation_is_attached`
- `test_permanent_missing_symbols_do_not_count_against_analyzable_coverage`
- `test_transient_missing_symbols_still_block_publish`
- `test_full_fetch_failure_preserves_existing_cache`
- `test_zero_results_preserve_existing_cache`
- `test_low_coverage_preserves_existing_cache`
- `test_kr_partial_source_failure_preserves_existing_cache`
- `test_us_partial_required_source_failure_preserves_existing_cache`
- `test_publish_failure_between_watchlist_and_current_rolls_back_generation`
- `test_success_replaces_current_and_watchlist`
- `test_main_continues_after_one_market_failure`

## Boundary Evidence

- Timestamp missing, malformed, future, wrong-scope, empty, count-inconsistent, and source-incomplete snapshots reject fresh scope.
- Partially null or row-inconsistent snapshot metadata also reject fresh scope.
- The valid fresh fixture is accepted by both KR and US leadership resolvers.
- KR KOSPI-only and US missing-NYSE-American provenance stop before liquidity processing and preserve existing operational files.
- An injected failure after watchlist replacement but before current replacement restores both prior files, removes the new dated snapshot, and leaves no `.tmp` files.
- `theme_hint=반도체` alone is review-only, customer-only industry phrases are not direct, while the MTSI structured semiconductor path remains direct.
- Future earnings, including a later timestamp on the same UTC date, are `distorted_or_missing`, unverified, and retain a negative age for diagnostics.
- Exact source allowlisting rejects lookalikes such as `unofficial` and `secondary estimate`.
- Frozen practical symbols `MTSI`, `SMTC`, `LSCC` appear in identical Markdown, Telegram, and TV-main order. Observation symbols are disjoint, and zero practical rows do not backfill Telegram Top.
- Builder-level fixture coverage uses real temporary files and the same canonical selector reaches Markdown, Telegram, and TradingView main outputs.
- The latest isolated live refresh reached KR `79.2%` and US `91.9%` coverage, produced a fresh weekly snapshot, and the daily dry-run consumed that snapshot without backfilling observation rows.

## Live Positive Path

One isolated live `--market both` weekly refresh completed successfully. It produced fresh weekly-universe output without touching the operational cache, and the following dry-run consumed that fresh snapshot:

- KR coverage: `79.2%`
- US coverage: `91.9%`
- Fresh snapshot: generated
- Practical count: `0`
- Observation backfill: none
- Telegram: skipped
- Google Drive: skipped
- Operational cache: unchanged

No Windows-only install path, credential, `.env`, Telegram identifier, rclone configuration, or operational result is part of this evidence.
