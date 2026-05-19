T20 STRATEGY — INBOX
====================

Drop new D02_NewMomentumBeta PDF reports into this folder.

Then ask Claude any of:
  - "process t20"
  - "update t20"
  - "extract t20 pdfs"

Claude will:
  1. Validate each PDF is a real T20 strategy report
  2. Skip dates already in the dashboard (deduplication)
  3. Extract date, strategy switch, and top 7 stocks
  4. Compute the 20th Bursa Malaysia trading day
  5. Update T20_Strategy_Dashboard.html and T20_Strategy_Data.csv
  6. Move the source PDF into "T20 strategy\" (the archive)
  7. Report what was processed / skipped / rejected

This README can be deleted whenever you like.
