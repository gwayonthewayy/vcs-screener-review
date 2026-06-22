# VCS Screener Briefing Logic V2 Review Bundle

This public repository contains a review-only bundle for a private project. It is not a runnable copy of the original repository.

## Review target

- Base branch: `main`
- Base commit: `107103b7520437ab9bacd4c76ed669feaa93f613`
- Feature branch: `fix/briefing-logic-v2`
- Feature commit: `13222251f9dd8d434f1071ffbc24d53b9c084215`
- Generated: 2026-06-22

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

The latest production run used `candidate_fallback_not_full_market`, so the fail-closed path was exercised and practical recommendations were zero. A fresh weekly-universe positive production path remains to be verified.
