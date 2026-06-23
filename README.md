# VCS Screener Briefing Logic V2 Review Bundle

This public repository contains a review-only bundle for a private project. It is not a runnable copy of the original repository.

## Review target

- Base branch: `main`
- Base commit: `107103b7520437ab9bacd4c76ed669feaa93f613`
- Feature branch: `fix/briefing-logic-v2`
- Feature commit: `1758fde0caca0434051ccc452fd1c926657d652a`
- Generated: 2026-06-23

## Files

- `REVIEW_BUNDLE.md`: background, architecture, gates, and known limitations
- `TEST_EVIDENCE.md`: regression test evidence
- `DRY_RUN_EVIDENCE.md`: KR/US/both dry-run evidence
- `BRIEFING_LOGIC_V2.md`: V2 selection policy
- `AUTOMATION.md`: automation call-path documentation
- `FINAL_REPORT.md`: single-file final report for the web reviewer
- `review.patch`: latest binary-capable diff from base to feature, excluding the older embedded review patch
- `WEB_REVIEW_PROMPT.md`: ready-to-use review request

## Security scope

The bundle intentionally excludes `.env`, credentials, Telegram identifiers, rclone configuration, `results/`, and `logs/`.

## Important limitation

Live positive path: PASS. The latest live isolated refresh completed with fresh weekly-universe coverage of KR `79.2%` and US `91.9%`. The daily dry-run consumed that fresh snapshot, practical recommendations remained zero, observation did not backfill practical, operational cache stayed unchanged, and Telegram/Drive delivery was skipped. Remaining review risks are the `ANALYSIS_READY_LIQUIDITY_RATIO=0.5` heuristic, Yahoo provider fragility, and the daily leadership label / product wording follow-up.
