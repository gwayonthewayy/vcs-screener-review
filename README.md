# VCS Screener Briefing Logic V2 Review Bundle

This public repository contains a review-only bundle for a private project. It is not a runnable copy of the original repository.

## Review target

- Base branch: `main`
- Base commit: `107103b7520437ab9bacd4c76ed669feaa93f613`
- Feature branch: `fix/briefing-logic-v2`
- Feature commit: `4c27f2e38e1261ffc70f27637b26930d1205e8b0`
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

The latest validation reached the live fresh weekly-universe positive path: KR coverage was `79.2%`, US coverage was `91.9%`, practical recommendations stayed at zero, and operational cache remained unchanged. A small analysis-ready coverage boundary regression was added afterward and is included in this bundle.
