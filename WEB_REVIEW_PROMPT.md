# Web GPT Review Prompt

Review this repository as a pre-merge code-review bundle for VCS Screener briefing logic V2.

Read `README.md`, `REVIEW_BUNDLE.md`, `TEST_EVIDENCE.md`, `DRY_RUN_EVIDENCE.md`, `BRIEFING_LOGIC_V2.md`, `AUTOMATION.md`, and finally `review.patch`. Treat `review.patch` as the authoritative production-code diff.

Focus on defects, behavioral regressions, unsafe fail-open behavior, call-path integration gaps, and missing tests. Do not assume claims in the evidence documents are correct without checking the patch.

Verify at minimum:

1. Daily KR/US/both automation actually calls V2 logic.
2. Missing practical flags, stale/fallback leadership, unverified earnings, future earnings, missing/below-EMA21 RS, ambiguous business classification, and high-risk biotech all fail closed.
3. Sector score 60 is a real boundary and VCS is not rewarded twice.
4. Weekly-universe refresh never overwrites a valid cache with an empty or failed refresh, including partial-source failures and publish rollback.
5. KR benchmarks use KOSPI/KOSDAQ by suffix; US uses QQQ with SPY fallback and records provenance.
6. Telegram, Markdown, and main TradingView outputs share one canonical practical selector and order.
7. Observation candidates cannot fill practical Top slots and practical/observation TV files cannot overlap.
8. N/A market data is not treated as bearish.
9. Environment precedence, zero-byte Telegram attachment skipping, and non-fatal rclone upload behavior do not expose secrets or break briefing completion.
10. The latest evidence set contains 44 passing tests, not 24.

Report findings first, ordered Critical/High/Medium/Low. For every finding provide file and line, triggering condition, impact, and a concrete fix direction. Then list merge blockers, non-blocking improvements, test gaps, and one verdict: `Ready to merge`, `Ready after fixes`, or `Not ready`.

Explicitly account for the remaining limitation that the fresh weekly-universe positive production path has not yet been demonstrated; the latest production result correctly produced zero practical candidates under candidate fallback scope.
