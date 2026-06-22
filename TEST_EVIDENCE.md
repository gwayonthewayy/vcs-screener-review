# Test Evidence

## Execution Commands

Executed in the repo root:

```bash
.venv/bin/python -m unittest discover -s tests -v
bash -n scripts/run_daily_briefing.sh
.venv/bin/python -m py_compile briefing_logic_v2.py candidate_enrichment_v2.py scripts/refresh_weekly_universe.py tests/test_briefing_logic_v2.py tests/test_weekly_universe_refresh.py scripts/build_market_briefing.py scripts/build_tradingview_watchlists.py scripts/build_telegram_summary.py build_deep_minervini_briefing.py build_whole_market_briefings.py
git diff --check
```

The weekly universe refresh was also attempted once:

```bash
.venv/bin/python scripts/refresh_weekly_universe.py --market both
```

That attempt failed with DNS resolution errors and preserved the existing cache.

## Result Summary

- `unittest`: 24 tests, all passed
- `bash -n`: passed
- `py_compile`: passed
- `git diff --check`: passed
- weekly refresh attempt: failed safely, cache preserved

## Full Test List

| Test | Result | What it guards |
| --- | --- | --- |
| `test_ambiguous_match_requires_review` | ok | Ambiguous business mapping stays review-only. |
| `test_ebay_and_dbd_are_not_ai_direct_from_software_word` | ok | Generic software wording does not imply AI directness. |
| `test_mtsi_is_direct_when_semiconductors_are_actually_leading` | ok | Real semiconductor evidence maps to direct business directness. |
| `test_missing_or_stale_earnings_fail_closed` | ok | Missing or stale earnings metadata fail closed. |
| `test_two_negative_opm_periods_are_loss_narrowing` | ok | Two negative OPM periods are treated as loss narrowing, not expansion. |
| `test_missing_market_data_is_neutral_not_bearish` | ok | Missing market data is neutral, not bearish. |
| `test_custom_watchlist_is_not_overridden` | ok | Manual watchlists are not silently rewritten. |
| `test_main_and_observation_tv_files_are_disjoint` | ok | Practical TV files and observation TV files do not overlap. |
| `test_rolling_watchlist_replaces_stale_dated_default` | ok | Legacy dated watchlists are replaced by rolling current files. |
| `test_rs_benchmark_mapping` | ok | KR/US benchmark mapping is correct. |
| `test_tv_export_fails_when_practical_schema_is_missing` | ok | TV export fail-closes if practical schema is missing. |
| `test_verified_direct_candidate_can_pass_end_to_end` | ok | A fully verified direct candidate passes the pipeline end-to-end. |
| `test_biotech_is_observation_only` | ok | QTTB/RLYB-like biotech names are observation-only. |
| `test_fresh_scope_and_sector_score_gate_practical` | ok | Fresh leadership scope and `sector_leadership_score >= 60` are required. |
| `test_ma50_minus_ten_percent_is_retained_but_not_immediate` | ok | MA50 -10% names remain in scan output but are not immediate buys. |
| `test_missing_practical_column_fails_closed` | ok | Missing practical schema fails closed. |
| `test_practical_top_does_not_backfill_observations` | ok | Practical Top is not padded with observation names. |
| `test_rs_unknown_or_below_ema21_is_not_practical` | ok | RS unknown or below EMA21 is not practical. |
| `test_sector_leadership_changes_recommend_score` | ok | Sector leadership contributes to score and stays bounded. |
| `test_full_fetch_failure_preserves_existing_cache` | ok | Weekly refresh failure does not overwrite existing cache. |
| `test_low_coverage_preserves_existing_cache` | ok | Low-coverage weekly refresh does not overwrite existing cache. |
| `test_main_continues_after_one_market_failure` | ok | One market failure does not stop the other market from being processed. |
| `test_success_replaces_current_and_watchlist` | ok | Valid refresh replaces the current snapshot and rolling watchlist. |
| `test_zero_results_preserve_existing_cache` | ok | Zero-result weekly refresh leaves existing cache intact. |

## Required Regression Checks

### Cache preservation

Covered by:

- `test_full_fetch_failure_preserves_existing_cache`
- `test_zero_results_preserve_existing_cache`
- `test_low_coverage_preserves_existing_cache`
- `test_success_replaces_current_and_watchlist`
- `test_main_continues_after_one_market_failure`

### Fresh / stale / fallback leadership scope

Covered by:

- `test_fresh_scope_and_sector_score_gate_practical`
- `test_verified_direct_candidate_can_pass_end_to_end`

The gate only accepts `weekly_liquid_full_market_fresh`.

### Sector score boundary

Covered by:

- `test_fresh_scope_and_sector_score_gate_practical`

It explicitly checks the `59.0 < 60.0` failure case and the `60.0` pass case.

### Practical / observation separation

Covered by:

- `test_practical_top_does_not_backfill_observations`
- `test_main_and_observation_tv_files_are_disjoint`
- `test_missing_practical_column_fails_closed`

### Windows / UTF-8 cleanup

The Linux server review path no longer depends on the Windows-only `install.cmd` artifact.
That file was removed from the branch, which avoids the confusing cross-platform encoding and line-ending path in the Linux review flow.

## Validation Notes

- `bash -n scripts/run_daily_briefing.sh` passed after the shell changes.
- `py_compile` passed for the V2 logic, weekly refresh, and downstream builders.
- `git diff --check` passed, so there are no whitespace or patch-format issues in the current working tree.
