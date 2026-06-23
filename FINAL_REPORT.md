# VCS Screener Final Review Report

This document is the single file the web reviewer should use for the final pass.
It summarizes the private feature branch, the validation result, and the published review bundle status.

1. 수정 전 HEAD
`1758fde0caca0434051ccc452fd1c926657d652a`

2. 수정한 3개 finding별 구현 내용
- 실적 source fail-open 제거: 부분 문자열 검사를 제거하고 exact allowlist만 허용하도록 바꿨다.
- weekly snapshot fail-open 제거: row-level metadata가 부분 null, 빈 문자열, 불일치, 비정상 count/coverage이면 fresh로 보지 않고 `invalid / candidate_fallback_not_full_market`로 처리한다.
- business directness fail-open 제거: 고객군 언급만으로 direct가 되지 않도록 하고, structured industry 또는 실제 제품/서비스 증거가 있어야 direct가 되게 했다.
- follow-up: `ANALYSIS_READY_LIQUIDITY_RATIO=0.5` 경계 regression을 추가해 analysis-ready denominator behavior를 고정했다.

3. 추가한 테스트 이름
- `test_unofficial_earnings_source_fails_closed`
- `test_secondary_estimate_source_fails_closed`
- `test_exact_official_and_provisional_sources_pass`
- `test_partial_null_market_row_is_not_fresh`
- `test_partial_blank_scope_row_is_not_fresh`
- `test_partial_null_snapshot_metadata_is_not_fresh`
- `test_inconsistent_snapshot_metadata_across_rows_is_not_fresh`
- `test_semiconductor_customer_mention_is_not_direct`
- `test_data_center_customer_mention_is_not_ai_direct`
- `test_logistics_for_chipmakers_requires_review`
- `test_builder_level_fresh_positive_fixture_keeps_order_and_disjoint_outputs`
- `test_analysis_ready_liquidity_ratio_boundary_counts_exactly_at_half_threshold`

4. 전체 테스트 개수와 통과 결과
- 총 `65`개 테스트 통과
- 실패 없음

5. `bash -n`, `py_compile`, `git diff --check` 결과
- `bash -n scripts/run_daily_briefing.sh` 통과
- `py_compile` 통과
- `git diff --check origin/main...HEAD` 통과

6. builder-level frozen positive fixture 결과
- 통과
- practical 순서: `MTSI`, `SMTC`, `LSCC`
- Markdown / Telegram Top / TradingView main 순서 동일
- practical / observation 교집합 0
- practical 0일 때 observation backfill 없음

7. live weekly refresh 검증 여부와 실패했다면 정확한 이유
- live positive path PASS
- KR coverage `79.2%`
- US coverage `91.9%`
- fresh weekly snapshot generated
- daily dry-run consumed fresh weekly watchlist
- practical `0`
- observation did not backfill practical
- Telegram/Drive no-send
- operational cache unchanged
- boundary regression was committed after the live PASS to pin the analysis-ready denominator heuristic

8. secret scan 결과
- 실제 `.env` 값 없음
- Telegram token/chat ID 없음
- rclone 인증정보 없음
- Google Drive credential 없음
- API key/private key 없음
- `results/`, `logs/` 운영 산출물 없음

9. `review.patch`의 `cmp` 및 SHA-256 검증 결과
- private diff와 byte-for-byte 동일
- SHA-256: `28817a5d77b6f2120ce8da4fc5397e791e98020c76e57b5f6a04e7542e3472a5`

10. 원본 feature 최종 commit SHA
`4c27f2e38e1261ffc70f27637b26930d1205e8b0`

11. 공개 review repo bundle status
- 공개 review repo는 최신 feature HEAD 기준으로 다시 갱신되었다.
- 순환 참조를 피하기 위해 이 문서에는 공개 repo 자기 자신의 tip SHA를 적지 않는다.

12. 두 repo의 clean 상태
- private repo: clean
- public review repo: clean after update

13. 남은 residual risk
- `ANALYSIS_READY_LIQUIDITY_RATIO=0.5` heuristic
- Yahoo provider fragility
- daily leadership label/product wording follow-up

14. `main` 병합 가능 여부 판단
- 병합 가능
