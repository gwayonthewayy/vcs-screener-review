# Server Automation

This repo keeps generated daily outputs out of Git. The committed sources are scripts, docs, and reusable examples only. Daily files belong under `results/`, logs under `logs/`, and any committed sample data should live in a future `fixtures/` or `examples/` folder.

## Run Modes

- `daily`: fast watchlist-based scan for routine KR/US briefing generation.
- `weekly`: broader universe scan to refresh candidates and catch new names that are not in the daily watchlists yet.
- `manual`: full scan on demand when explicitly requested.

Daily mode uses:

```bash
scripts/run_daily_briefing.sh --market both
```

Market-specific runs are supported:

```bash
scripts/run_daily_briefing.sh --market us
scripts/run_daily_briefing.sh --market kr
scripts/run_daily_briefing.sh --market both
```

- `--market us`: US scan, US briefing, US TradingView files, and `[VCS US Briefing]` Telegram package.
- `--market kr`: KR scan, KR briefing, KR TradingView files, and `[VCS KR Briefing]` Telegram package.
- `--market both`: KR/US scans, integrated briefing, KR/US TradingView files, and `[VCS Integrated Briefing]` Telegram package.
- Default is `both`.

The default daily watchlists are:

```text
watchlists/kr_stage1_20260312_022553.txt
watchlists/us_stage1_20260312_022553.txt
```

Weekly/manual full scans should omit `--kr-watchlist` and `--us-watchlist`, or use a much broader universe input. Because full scans can take much longer, run them outside the daily automation window.

## Environment

Create a local `.env` from `.env.example` when notifications are needed. Never commit `.env`.

```bash
cp .env.example .env
```

Telegram variables:

```text
TELEGRAM_BOT_TOKEN=
TELEGRAM_CHAT_ID=
```

The daily script sends Telegram notifications only when both variables are present. Secrets may also be exported by the shell, systemd, or cron environment instead of `.env`. When a variable is set both in the shell and in `.env`, the shell value wins.

Successful daily runs send:

- A Telegram-readable summary with market grade, KR/US candidate counts, sector leadership, top candidates, and 2-3 interpretation lines.
- The final integrated briefing markdown file.
- The final integrated briefing DOCX file.
- KR and US TradingView sections text files.

The same summary is saved locally with the market in the filename:

```text
results/YYYYMMDD_CODEX/YYYYMMDD_CODEX_US_telegram_summary.txt
results/YYYYMMDD_CODEX/YYYYMMDD_CODEX_KR_telegram_summary.txt
results/YYYYMMDD_CODEX/YYYYMMDD_CODEX_BOTH_telegram_summary.txt
```

Telegram send failures are logged but do not fail the whole briefing run.

Google Drive upload is optional and uses `rclone`, configured outside this repo. It runs after the Telegram summary and document sends. Upload failures are logged with `[gdrive]` but do not fail the briefing run.

```text
ENABLE_GDRIVE_UPLOAD=0
GDRIVE_RCLONE_REMOTE=gdrive
GDRIVE_UPLOAD_PATH=VCS_Briefings
GDRIVE_UPLOAD_EARNINGS=0
```

Upload target:

```text
${GDRIVE_RCLONE_REMOTE}:${GDRIVE_UPLOAD_PATH}/YYYYMMDD_CODEX/
```

Default upload package by market:

- `--market us`: US Telegram summary txt, US briefing md/docx, US TradingView sections txt, US TradingView upload txt.
- `--market kr`: KR Telegram summary txt, KR briefing md/docx, KR TradingView sections txt, KR TradingView upload txt.
- `--market both`: integrated Telegram summary txt, integrated briefing md/docx, KR/US TradingView sections txt, KR/US TradingView upload txt.

If `GDRIVE_UPLOAD_EARNINGS=1`, the market-matching `실적양호` CSV/XLSX files are uploaded too.

Skip behavior:

- `ENABLE_GDRIVE_UPLOAD=0`: skip quietly with a single log line.
- `ENABLE_GDRIVE_UPLOAD=1` and `rclone` missing: warn and skip.
- `ENABLE_GDRIVE_UPLOAD=1` and the configured remote is missing: warn and skip.
- When enabled, the script first creates the dated Drive folder with `rclone mkdir` and then uploads files into that folder.
- Upload failures for individual files are logged but do not change the briefing exit code.

Set up `rclone` interactively on the server:

```bash
rclone config
rclone listremotes
rclone lsd gdrive:
```

Use the remote name from `rclone listremotes` without the trailing colon in `.env`, for example `GDRIVE_RCLONE_REMOTE=gdrive`.

The Drive upload step can be smoke-tested without rerunning scans:

```bash
ENABLE_GDRIVE_UPLOAD=0 bash scripts/run_daily_briefing.sh --market us --gdrive-upload-only
ENABLE_GDRIVE_UPLOAD=1 GDRIVE_RCLONE_REMOTE=missing_remote bash scripts/run_daily_briefing.sh --market us --gdrive-upload-only
```

The upload-only mode uses the expected files in `results/YYYYMMDD_CODEX/` for the selected market and exits after the Drive upload step.

## Smoke Test

Run this before enabling any schedule:

```bash
scripts/smoke_test.sh
```

It checks the venv, Python version, required package imports, `full_market_scan.py --help`, and write access to `results/` and `logs/`.

## Scheduling

Cron is registered for the current server user. The server timezone is UTC, and
`CRON_TZ=Asia/Seoul` did not reliably affect schedule interpretation on this
host during the 2026-06-18 check. The active crontab therefore uses UTC schedule
times while each command still sets `TZ=Asia/Seoul` so script timestamps, result
folders, and briefing dates stay aligned with KST.

Target schedule in `Asia/Seoul`:

- US briefing: weekday `07:10`, after the US close.
- KR briefing: weekday `16:10`, after the KR close.
- Integrated briefing: weekday `16:30`, after the KR close and after both market-specific packages are available.

Registered cron block:

```cron
# BEGIN VCS Screener daily automation
# Server timezone is UTC. Schedule below is UTC, targeting KST weekdays.
# KST 07:10 Mon-Fri = UTC 22:10 Sun-Thu.
10 22 * * 0-4 cd /home/gyu123/projects/vcs-screener && TZ=Asia/Seoul /bin/bash scripts/run_daily_briefing.sh --market us
# KST 16:10 Mon-Fri = UTC 07:10 Mon-Fri.
10 7 * * 1-5 cd /home/gyu123/projects/vcs-screener && TZ=Asia/Seoul /bin/bash scripts/run_daily_briefing.sh --market kr
# KST 16:30 Mon-Fri = UTC 07:30 Mon-Fri.
30 7 * * 1-5 cd /home/gyu123/projects/vcs-screener && TZ=Asia/Seoul /bin/bash scripts/run_daily_briefing.sh --market both
# END VCS Screener daily automation
```

Do not add `CRON_TZ=Asia/Seoul` back unless it is re-tested on the target host.
If the server timezone is later changed to `Asia/Seoul`, update these cron times
back to the KST clock times after confirming `date`, `timedatectl`, and a
short Telegram cron test.

Manual inspection command:

```bash
crontab -l
```

Systemd timer is also fine if you prefer managed logs/restarts. Use `Environment=TZ=Asia/Seoul`, `WorkingDirectory=/home/gyu123/projects/vcs-screener`, and `ExecStart=/home/gyu123/projects/vcs-screener/scripts/run_daily_briefing.sh --market us` or the matching market option.

## Output Handling

Priority order:

1. Save everything locally under `results/YYYYMMDD_CODEX`.
2. Send Telegram summary plus final briefing and TradingView attachments.
3. Optionally upload the same summary/briefing/TradingView package to Google Drive via `rclone`.

The script writes its log to:

```text
logs/YYYYMMDD_daily.log
```

## Recommendation Logic Notes

The v2 briefing pipeline now applies the practical-vs-observation split in code and exports. The current contract is:

- `상위 후보` and `실전 추천 후보` are distinct.
- Telegram Top 5 uses only practical recommendation candidates.
- High-risk biotech, deficit, and distorted-OPM names stay in observation-only buckets.
- Biotech names default to `비주도/고위험 / 바이오텍`.
- RS must be confirmed and above its moving average for immediate-buy eligibility.
- VCS is standardized at `45`.
- Briefings state whether they come from the daily watchlist or a whole-market leadership scan.
- The recommendation order is explicit: `주도섹터 -> 실적 -> OPM -> VCS -> RS -> 자리/이격 -> 수급`.

If this policy changes again, update `docs/briefing_logic_v2.md` and the selection tests first, then refresh the automation notes.
