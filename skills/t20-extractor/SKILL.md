---
name: t20-extractor
description: Extract data from daily T20 strategy PDF reports (Stock Insight Malaysia / Bursa Malaysia momentum-beta style reports) and update the Stock Insight T20 dashboard. Use this skill whenever the user says "process t20", "update t20", "extract t20", "new t20 pdf", "update the dashboard", drops PDFs in the T20 inbox folder, or attaches a PDF that contains the ranked table header "Code Stock Liquidity Close Sector MarketCap" and a "Strategy Switch:" line — even if they don't explicitly name the skill. Use this skill for the full pipeline: format validation, deduplication against the existing dashboard, extracting the top-7 ranked stocks with full row data (code, stock, liquidity, close, sector, market cap), computing the 20th Bursa Malaysia trading day using the gazetted 2026 holiday calendar, updating the dashboard HTML and CSV in chronological order, and archiving the source PDF.
---

# T20 Strategy Extractor

End-to-end pipeline for processing daily T20 strategy PDFs and updating the user's Stock Insight T20 dashboard.

The PDFs come from a third-party service. The strategy name printed in the report (e.g. `D02_NewMomentumBeta`) may change over time — **do not rely on it for validation**. Use structural markers instead: the table header, the `Strategy Switch:` line, and the report date.

## Fixed paths

- **Inbox** (new PDFs land here): `D:\08 project\Stock Insight - T20\inbox\`
- **Archive** (where processed PDFs are moved to): `D:\08 project\Stock Insight - T20\T20 strategy\`
- **Dashboard**: `D:\08 project\Stock Insight - T20\T20_Strategy_Dashboard.html`
- **CSV**: `D:\08 project\Stock Insight - T20\T20_Strategy_Data.csv`

## Step-by-step

### 1. List the inbox

Use `Glob` with pattern `D:\08 project\Stock Insight - T20\inbox\*.pdf`. Ignore non-PDF files (e.g. README). If the result is empty, tell the user "Inbox is empty — no new T20 PDFs to process" and stop.

If the user instead attached a PDF directly to the chat (path under `uploads/`), treat that single file as the work item. After processing, ask whether to also copy it into the archive folder (you can't move it from `uploads/` — only copy).

### 2. Validate each PDF (structural — NOT name-based)

`Read` the PDF and confirm ALL of these markers appear in the extracted text:

1. The exact header row: `Code Stock Liquidity Close Sector MarketCap`
2. A line matching the regex `Strategy Switch:\s*(ON \(Bullish\)|OFF \(Bearish\))`
3. A date matching `\d{4}-\d{2}-\d{2}` appearing on or adjacent to the same line as the Strategy Switch line (typically the report date is shown right before "Strategy Switch:" — often on the previous line, after the strategy name)
4. At least 7 data rows beneath the header

If any check fails, record the filename + the failed check and skip that file. Do NOT validate by checking for any particular strategy name — the name is not under the user's control and may change.

### 3. Deduplicate against the dashboard

`Read` `D:\08 project\Stock Insight - T20\T20_Strategy_Dashboard.html` and search for `date:"<the new date>"` inside the `REPORTS` array. If present, mark as duplicate and skip extraction for that file — but still archive it in step 8 (since it's a known good file).

### 4. Extract the data

From the PDF text:

- **Report date** → the `YYYY-MM-DD` that appears immediately before or on the same line as `Strategy Switch:`. (In the standard layout this is the line `<strategy_name> YYYY-MM-DD` directly above `Strategy Switch:`.)
- **Day of week** → derive from the date.
- **Switch** → `"ON"` if the line says `ON (Bullish)`, else `"OFF"`.
- **Strategy name** → the token(s) on the same line as the report date, before the date. Capture this for the dashboard subtitle, but never use it for validation.
- **Top 7 stocks** → the first 7 rows beneath the `Code Stock Liquidity Close Sector MarketCap` header. Each row has 6 columns:
  - `code` — 4 alphanumeric characters (e.g. `5210`, `0138`)
  - `stock` — single whitespace-free token (may contain `&`, e.g. `ARMADA`, `E&O`, `F&N`)
  - `liq` — number ending in `m` or `b` (e.g. `13.927m`)
  - `close` — decimal number (e.g. `0.375`)
  - `sector` — 1–4 words, may contain `&` (e.g. `Energy`, `Health Care`, `Real Estate Investment Trusts`, `Industrial Products & Services`)
  - `mc` — number ending in `m` or `b` (e.g. `2.223b`)

  Parse trick (robust to multi-word sectors): tokenize each row by whitespace. The **first token** is `code`, **second** is `stock`, **third** is `liq`, **fourth** is `close`, **last** is `mc`. Everything between `close` and `mc` (joined with single spaces) is `sector`.

### 5. Compute the 20th trading day

Starting from the report date, count forward one calendar day at a time. A day counts as 1 trading day only if it is not Saturday, not Sunday, and not in the holiday list below. Stop when the count reaches 20.

**Bursa Malaysia 2026 trading-day-blocked holidays** (the gazetted federal list — Sunday-falling holidays are reflected by their Monday-observed equivalents):

| Date | Holiday |
|------|---------|
| 2026-01-01 (Thu) | New Year's Day |
| 2026-02-02 (Mon) | Thaipusam (observed) |
| 2026-02-17 (Tue) | Chinese New Year Day 1 |
| 2026-02-18 (Wed) | Chinese New Year Day 2 |
| 2026-03-23 (Mon) | Hari Raya Puasa Day 2 (observed) |
| 2026-05-01 (Fri) | Workers' Day |
| 2026-05-27 (Wed) | Hari Raya Haji |
| 2026-06-01 (Mon) | Yang Dipertuan Agong's Birthday / Wesak (observed) |
| 2026-06-17 (Wed) | Awal Muharram |
| 2026-08-25 (Tue) | Birthday of Prophet Muhammad |
| 2026-08-31 (Mon) | National Day (Merdeka) |
| 2026-09-16 (Wed) | Malaysia Day |
| 2026-11-09 (Mon) | Deepavali (observed) |
| 2026-12-25 (Fri) | Christmas Day |

For PDFs from 2027+, ask the user for the new year's Bursa holiday list before computing — don't extrapolate.

### 6. Update the dashboard

`Edit` `T20_Strategy_Dashboard.html`. Find the chronologically-last entry in the `REPORTS = [...]` array whose date is ≤ the new date, then insert the new entry on the line immediately after it. If the new date is earlier than every existing entry, insert at the top. Match the existing single-line per-row formatting exactly.

Template:
```javascript
  {date:"YYYY-MM-DD", dow:"DayName", sw:"ON",  t20:"YYYY-MM-DD", top7:[
    {code:"XXXX",stock:"NAME",  liq:"X.XXXm", close:"X.XX",  sector:"Sector",   mc:"X.XXXb"},
    ... 7 rows total ...
  ]},
```

Whitespace/padding in the row literals is cosmetic — the dashboard parses by field name. The streak is computed at runtime; do not include it.

### 7. Update the CSV

`Edit` `T20_Strategy_Data.csv`. Insert the new row in chronological order:
```
YYYY-MM-DD,DayOfWeek,SwitchLabel,20th-Trading-Day,Top1,Top2,Top3,Top4,Top5,Top6,Top7
```
Where `SwitchLabel` is exactly `ON (Bullish)` or `OFF (Bearish)`. The top stocks are just the stock names (no other columns).

### 8. Archive the PDF

Move the file from inbox to the archive. If `mcp__workspace__bash` is available:
```bash
mv "/sessions/<session>/mnt/Stock Insight - T20/inbox/<file>.pdf" \
   "/sessions/<session>/mnt/Stock Insight - T20/T20 strategy/<file>.pdf"
```
The session-specific mount path is shown in the system prompt's "Shell access" section.

If bash is unavailable, tell the user: "Bash is offline in this session — please move `<file>.pdf` from `inbox\` into `T20 strategy\` manually."

### 9. Report

End with a concise summary, e.g.:
> Processed 2 new T20 reports:
> - **2026-05-19** (Tue) — ON (Bullish), 20th trading day = 2026-06-19
> - **2026-05-20** (Wed) — ON (Bullish), 20th trading day = 2026-06-22
>
> Skipped 1 duplicate (2026-05-18). Rejected 0 invalid files.
>
> [Open dashboard](computer://D:\08 project\Stock Insight - T20\T20_Strategy_Dashboard.html)

## Edge cases

- **Stock symbol contains `&`** (e.g. `E&O`, `F&N`): the stock is still a single whitespace-free token, so the tokenization rule in step 4 handles it correctly.
- **Sector contains `&`** (e.g. `Industrial Products & Services`): handled by the "everything between `close` and `mc`" rule.
- **Filename unreliable**: filenames may be `YYYYMMDD.pdf`, `DOC-YYYYMMDD-WA0000..pdf`, or anything else. Always use the date extracted from the PDF *content*, not the filename.
- **Multiple PDFs arriving for the same date**: keep the first valid one, treat the others as duplicates.
- **Out-of-order arrivals** (e.g. a back-dated PDF arrives after a newer one): still insert at the correct chronological position. The dashboard re-computes streaks based on array order, so the order matters.
- **PDF has fewer than 7 data rows**: reject as malformed (validation step 4).
- **Strategy name changed**: extract and use the new name, but do not treat the change as an error. If the dashboard's displayed strategy name (the subtitle next to "Strategy Switch") is hardcoded HTML, update it to read from the latest entry's `strategy` field.

## Why this design

- **Inbox + archive pattern** keeps "to-do" and "done" visually separable so the user can confirm at a glance that processing happened.
- **Structural validation** (header row + Strategy Switch line + date) is stable across vendor changes to titling, branding, or strategy naming.
- **Deduplication on content date**, not filename, means the user can drop the same file twice without corrupting the dashboard.
- **Insertion in chronological order** preserves the streak computation logic — the dashboard relies on array order, not on stored streak values.
