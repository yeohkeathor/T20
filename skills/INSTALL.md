# Installing the t20-extractor skill

The skill lives in `t20-extractor\SKILL.md` (sibling of this file).

To activate it in Cowork, copy the `t20-extractor` folder to your Claude skills directory:

**Source** (in this workspace):
```
D:\08 project\Stock Insight - T20\skills\t20-extractor\
```

**Destination** (Claude application data):
```
C:\Users\Keat Hor\AppData\Roaming\Claude\local-agent-mode-sessions\skills-plugin\5c6dc2f5-2ef9-4e9d-bcb2-224512dae027\26971c29-5037-4718-95e2-24e25247b24b\skills\t20-extractor\
```

PowerShell one-liner:
```powershell
Copy-Item -Recurse -Force `
  "D:\08 project\Stock Insight - T20\skills\t20-extractor" `
  "C:\Users\Keat Hor\AppData\Roaming\Claude\local-agent-mode-sessions\skills-plugin\5c6dc2f5-2ef9-4e9d-bcb2-224512dae027\26971c29-5037-4718-95e2-24e25247b24b\skills\"
```

After copying, restart the Cowork session (or just open a new chat). The skill will appear in `available_skills` and Claude will auto-trigger it on phrases like "process t20", "update t20", or when you attach a T20 PDF.

If the GUID-shaped path segments differ on your machine, navigate to `C:\Users\<you>\AppData\Roaming\Claude\local-agent-mode-sessions\skills-plugin\` and find the matching `skills\` folder.
