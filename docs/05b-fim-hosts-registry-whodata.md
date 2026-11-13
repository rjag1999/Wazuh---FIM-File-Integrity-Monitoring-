# 5b. FIM, the real-world way: hosts file + Run registry key (who-data)

**Status: ⏳ In progress**

Goal: watch two common attacker targets on win-131 and record **who** changed them.

| Target | Why attackers touch it | Mode |
|---|---|---|
| `C:\Windows\System32\drivers\etc\hosts` | Redirects domains (e.g. a bank site) to an attacker IP | `whodata` (real time + user/process) |
| `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` | Persistence: program runs at every boot | Scheduled scan (registry has no real-time mode) |

## Plan
1. Check which defaults already cover these paths (`Select-String` on ossec.conf).
2. Set the hosts entry to `whodata="yes" report_changes="yes"`; set the Run key to `report_changes="yes"`.
3. Restart WazuhSvc; confirm that the who-data engine started (ossec.log).
4. Attack simulation: add a fake line to hosts; add a fake `Run` value.
5. Registry: force a scan from the manager (`agent_control -r -u 001`).
6. Check the alerts: `audit.process.name`, `audit.user.name`, `syscheck.diff`.
7. Clean up: revert hosts and remove the Run value.

_Results will be added here once completed._
