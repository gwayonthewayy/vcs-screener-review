# Dry Run Evidence

Source artifact:

- `results/logic_v2_audit/20260619_CODEX_dryrun/20260619_CODEX_dryrun_report.md`

The report was generated from the current repository outputs for the `--market us`, `--market kr`, and `--market both` briefing paths.

## Commands Referenced by the Report

- `bash scripts/run_daily_briefing.sh --market us`
- `bash scripts/run_daily_briefing.sh --market kr`
- `bash scripts/run_daily_briefing.sh --market both`

## Weekly Universe Status

The weekly current files are absent locally:

| file | exists | scope shown in report |
| --- | --- | --- |
| `universes/kr_liquid_current.csv` | no | `candidate_fallback_not_full_market` |
| `universes/us_liquid_current.csv` | no | `candidate_fallback_not_full_market` |

One live refresh attempt was made:

```bash
.venv/bin/python scripts/refresh_weekly_universe.py --market both
```

It failed safely with DNS resolution errors for both upstream data sources:

- `kind.krx.co.kr`
- `www.nasdaqtrader.com`

The script logged that the existing cache was preserved.

## Dry-Run Summary

| market | candidate rows in report | practical candidates | observation rows | leadership scope | practical reason |
| --- | ---: | ---: | ---: | --- | --- |
| KR | 7 | 0 | 7 | `candidate_fallback_not_full_market` | scope gate closed by design |
| US | 48 | 0 | 48 | `candidate_fallback_not_full_market` | scope gate closed by design |
| BOTH | integrated practical top 0 / 0 | 0 | 0 | fallback on both markets | no fresh weekly leadership |

## Why Practical Count Is 0

The exact reason is not a scoring failure.

The practical gate closes because the leadership universe scope is fallback, not fresh:

- `leadership_universe_scope != weekly_liquid_full_market_fresh`
- weekly current cache is missing locally
- the safe refresh attempt failed and preserved the prior cache state

This is a fail-closed outcome, not a fake empty-file outcome.

## Candidate Alignment

### Markdown

- Practical Top in KR: empty
- Practical Top in US: empty
- Integrated practical Top: empty

### Telegram

The report shows Telegram practical content aligned with the canonical selector.
Observation text is kept in separate sections and does not promote observation names into practical Top.

### TradingView

- Practical TV main symbols: `∅`
- Practical TV observation symbols: `∅`
- Main / observation intersection: `∅`

That means there is no leakage from the practical selector into the wrong TV file.

## Column Evidence - KR

| column | exists | non-null | sample up to 5 |
| --- | --- | ---: | --- |
| `leadership_universe_scope` | yes | 7 | `candidate_fallback_not_full_market` |
| `sector_leadership_score` | yes | 7 | `62.69`, `71.88`, `62.41`, `39.53` |
| `business_directness` | yes | 7 | `direct` |
| `needs_sector_review` | yes | 7 | `False` |
| `earnings_quarter` | yes | 7 | `2026Q1` |
| `earnings_release_date` | yes | 7 | `2026-05-11`, `2026-05-13`, `2026-05-15`, `2026-05-14`, `2026-05-12` |
| `earnings_source_type` | yes | 7 | `official` |
| `earnings_verified` | yes | 7 | `True` |
| `earnings_quality` | yes | 7 | `profitable_expansion`, `turnaround_positive`, `profitable_slowdown` |
| `opm_latest_q` | yes | 7 | `40.65832526605495`, `23.01863600581545`, `17.54897040918415`, `37.18774031424255`, `18.23030051666868` |
| `opm_roll4q` | yes | 7 | `35.79432881006059`, `12.771977838744045`, `8.894961015801819`, `22.8635809450271`, `8.807439158794029` |
| `opm_roll12q` | yes | 7 | `25.24070401779174`, `0.7415602889216886`, `3.505459147639354`, `10.367862201352686`, `2.671029554486072` |
| `rs_benchmark` | yes | 7 | `KOSDAQ` |
| `rs_benchmark_source` | yes | 7 | `yfinance:^KQ11` |
| `rs_score` | yes | 7 | `98.0`, `91.0`, `97.0`, `99.0`, `95.0` |
| `rs_above_ma` | yes | 7 | `True`, `False` |
| `rs_confidence` | yes | 7 | `high` |
| `recommend_score` | yes | 0 | `∅` |
| `observation_score` | yes | 7 | `78.4685`, `77.2792`, `76.8555`, `72.3523`, `69.0533` |
| `is_practical_recommendation` | yes | 7 | `False` |
| `recommend_reason` | yes | 7 | `주도섹터 scope 미검증(candidate_fallback_not_full_market)`, `RS Line EMA21 하방`, `MA50 회복 대기` |

## Column Evidence - US

| column | exists | non-null | sample up to 5 |
| --- | --- | ---: | --- |
| `leadership_universe_scope` | yes | 48 | `candidate_fallback_not_full_market` |
| `sector_leadership_score` | yes | 37 | `88.37`, `67.15`, `68.47`, `44.63`, `69.06` |
| `business_directness` | yes | 48 | `direct`, `peripheral`, `adjacent`, `unrelated` |
| `needs_sector_review` | yes | 48 | `False`, `True` |
| `earnings_quarter` | yes | 48 | `2026Q1` |
| `earnings_release_date` | yes | 48 | `2026-05-06`, `2026-05-05`, `2026-05-07`, `2026-04-29`, `2026-02-05` |
| `earnings_source_type` | yes | 48 | `official` |
| `earnings_verified` | yes | 48 | `True`, `False` |
| `earnings_quality` | yes | 48 | `profitable_expansion`, `profitable_slowdown`, `distorted_or_missing`, `loss_narrowing` |
| `opm_latest_q` | yes | 47 | `16.52848155696844`, `31.4903171484704`, `100.3021047423308`, `10.019813189923578`, `61.56951664638868` |
| `opm_roll4q` | yes | 48 | `15.548527173642285`, `20.504320575310707`, `67.92882834082216`, `9.41816659690247`, `46.419872861747294` |
| `opm_roll12q` | yes | 48 | `14.440959352965155`, `15.133278666703914`, `62.09135856077543`, `9.129107981220658`, `45.76741817217767` |
| `rs_benchmark` | yes | 48 | `QQQ` |
| `rs_benchmark_source` | yes | 48 | `yfinance:QQQ` |
| `rs_score` | yes | 48 | `95.0`, `72.0`, `63.0`, `91.0`, `74.0` |
| `rs_above_ma` | yes | 48 | `True`, `False` |
| `rs_confidence` | yes | 48 | `high` |
| `recommend_score` | yes | 0 | `∅` |
| `observation_score` | yes | 48 | `77.803`, `75.9314`, `74.5039`, `74.2032`, `72.0513` |
| `is_practical_recommendation` | yes | 48 | `False` |
| `recommend_reason` | yes | 48 | `주도섹터 scope 미검증(candidate_fallback_not_full_market)`, `RS Line EMA21 하방`, `고위험 바이오텍`, `사업 직접성 미검증` |

## Integrated Notes

- Markdown practical Top symbols: `∅`
- Telegram practical Top symbols: `∅`
- TV main / observation intersection: `∅`
- QTTB in main TV: `no`
- QTTB in observation TV: `no`
- RLYB in main TV: `no`
- RLYB in observation TV: `no`

The evidence set therefore shows a strict fail-closed stance rather than a fake promotion of observation names.
