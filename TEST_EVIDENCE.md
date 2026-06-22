# Test Evidence

## Final Review Remediation Run

Commands:

```bash
.venv/bin/python -m unittest discover -s tests -v
bash -n scripts/run_daily_briefing.sh
.venv/bin/python -m py_compile briefing_logic_v2.py candidate_enrichment_v2.py scripts/refresh_weekly_universe.py kr_market_sector_leadership.py us_sector_leadership.py weekly_universe_cache.py full_market_scan.py
git diff --check origin/main...HEAD
```

Result: 44 tests passed. Bash syntax, Python compilation, and diff whitespace checks passed.

One bounded live weekly refresh was also attempted:

```bash
.venv/bin/python scripts/refresh_weekly_universe.py --market both
```

That attempt produced no upstream response within the bounded run and was terminated safely. The existing cache state was preserved.

## Test Inventory

### Refresh provenance and transaction safety

- `test_kr_partial_source_failure_preserves_existing_cache`
- `test_us_partial_required_source_failure_preserves_existing_cache`
- `test_full_fetch_failure_preserves_existing_cache`
- `test_zero_results_preserve_existing_cache`
- `test_low_coverage_preserves_existing_cache`
- `test_success_replaces_current_and_watchlist`
- `test_publish_failure_between_watchlist_and_current_rolls_back_generation`
- `test_main_continues_after_one_market_failure`

### Weekly cache validation

- `test_missing_timestamp_column_is_not_fresh`
- `test_unparseable_timestamp_is_not_fresh`
- `test_future_timestamp_is_not_fresh`
- `test_wrong_universe_scope_is_not_fresh`
- `test_empty_weekly_csv_is_not_fresh`
- `test_invalid_count_or_source_metadata_is_not_fresh`
- `test_invalid_coverage_metadata_is_not_fresh`
- `test_failed_source_status_metadata_is_not_fresh`
- `test_valid_fresh_snapshot_is_accepted`
- `test_valid_old_snapshot_is_stale_not_fresh`
- `test_kr_fresh_scope_markdown_title_is_full_market`

### Business and earnings

- `test_ebay_and_dbd_are_not_ai_direct_from_software_word`
- `test_mtsi_is_direct_when_semiconductors_are_actually_leading`
- `test_ambiguous_match_requires_review`
- `test_theme_hint_semiconductor_alone_is_not_direct`
- `test_two_negative_opm_periods_are_loss_narrowing`
- `test_missing_or_stale_earnings_fail_closed`
- `test_future_release_date_fails_closed`
- `test_same_day_future_release_time_fails_closed`
- `test_quarter_after_release_date_fails_closed`

### Recommendation, market, and output contracts

- `test_biotech_is_observation_only`
- `test_rs_unknown_or_below_ema21_is_not_practical`
- `test_ma50_minus_ten_percent_is_retained_but_not_immediate`
- `test_sector_leadership_changes_recommend_score`
- `test_fresh_scope_and_sector_score_gate_practical`
- `test_practical_top_does_not_backfill_observations`
- `test_missing_practical_column_fails_closed`
- `test_missing_market_data_is_neutral_not_bearish`
- `test_verified_direct_candidate_can_pass_end_to_end`
- `test_rs_benchmark_mapping`
- `test_rolling_watchlist_replaces_stale_dated_default`
- `test_custom_watchlist_is_not_overridden`
- `test_main_and_observation_tv_files_are_disjoint`
- `test_tv_export_fails_when_practical_schema_is_missing`
- `test_fresh_positive_fixture_has_identical_markdown_telegram_and_tv_order`
- `test_zero_practical_candidates_do_not_backfill_telegram_top`

## Boundary Evidence

- Timestamp missing, malformed, future, wrong-scope, empty, count-inconsistent, and source-incomplete snapshots reject fresh scope.
- The valid fresh fixture is accepted by both KR and US leadership resolvers.
- KR KOSPI-only and US missing-NYSE-American provenance stop before liquidity processing and preserve existing operational files.
- An injected failure after watchlist replacement but before current replacement restores the prior generation and leaves no `.tmp` files.
- `theme_hint=반도체` alone is review-only, while the MTSI structured semiconductor path remains direct.
- Future earnings, including a later timestamp on the same UTC date, are `distorted_or_missing`, unverified, and retain a negative age for diagnostics.
- Frozen practical symbols `MTSI`, `SMTC`, `LSCC` appear in identical Markdown, Telegram, and TV-main order.
- Observation symbols are disjoint, and zero practical rows do not backfill Telegram Top.

## Validation Notes

- `bash -n scripts/run_daily_briefing.sh` passed.
- `py_compile` passed for the V2 logic, weekly refresh, and downstream builders.
- `git diff --check origin/main...HEAD` passed, so the final bundle is whitespace-clean.
