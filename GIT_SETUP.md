# Git setup — run once

Open PowerShell (or your terminal of choice), cd into the workspace, then run:

```powershell
cd "D:\08 project\Stock Insight - T20"

git init
git config user.name  "Keat Hor"
git config user.email "yeohkeathor@gmail.com"

git add .
git commit -m "Initial Stock Insight T20 dashboard + extractor skill

- Dashboard with date-picker calendar (green/red by status), top-7 stocks
  table with hyperlinks, dynamic strategy name
- CSV of all extracted reports through 2026-05-18
- 27 historical T20 strategy PDFs in T20 strategy/
- t20-extractor skill (validates by structural markers, not strategy name;
  full 20th-trading-day calc against 2026 Bursa Malaysia holidays;
  inbox-to-archive workflow with deduplication)
- INSTALL.md for activating the skill in Cowork"
```

After this initial commit, every subsequent change (e.g. when Claude processes a new T20 PDF) can be committed with:

```powershell
git add .
git commit -m "Add T20 report YYYY-MM-DD"
```

If you'd like, push to a remote (GitHub/GitLab/Bitbucket) by adding it later:

```powershell
git remote add origin <your-repo-url>
git push -u origin main
```
