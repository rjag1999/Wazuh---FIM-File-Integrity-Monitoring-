# 5. File Integrity Monitoring (FIM): Windows Desktop

**Status: ✅ Done (2026-11-13)**

Goal: alert in real time when files on `C:\Users\Clientew10\Desktop` are
created, modified or deleted, and show what changed.

## How FIM works (short version)

The Wazuh agent's **syscheck** module records a baseline of each monitored file:
size, permissions, owner, modification time and MD5/SHA1/SHA256 hashes. If any
of those change, the agent sends an event to the manager, which raises an alert.

- By default, syscheck only re-scans every **12 hours** (`<frequency>43200</frequency>`).
- `realtime="yes"` uses Windows file-change notifications, so the alert comes within seconds.
- `report_changes="yes"` keeps a copy of each text file, so the alert includes a
  **diff** of the exact lines that changed (`syscheck.diff`).

## 1. Configure the agent (win-131)

1. Open Notepad **as Administrator** and open
   `C:\Program Files (x86)\ossec-agent\ossec.conf`
   (set the file filter to *All files*).
2. Inside the `<syscheck>` block, add:
   ```xml
   <directories realtime="yes" report_changes="yes">C:\Users\Clientew10\Desktop</directories>
   ```
3. Restart the agent (PowerShell as Administrator):
   ```powershell
   Restart-Service WazuhSvc
   Get-Service WazuhSvc     # Status should be Running
   ```

> If the XML has a typo, `WazuhSvc` won't start. Check the end of
> `C:\Program Files (x86)\ossec-agent\ossec.log` for the error.

## 2. Verify the agent loaded the config

```powershell
# Is the line in the config?
Select-String -Path "C:\Program Files (x86)\ossec-agent\ossec.conf" -Pattern "Desktop"

# Is the agent actually monitoring the path?
Select-String -Path "C:\Program Files (x86)\ossec-agent\ossec.log" -Pattern "desktop" | Select-Object -Last 5
```

Expected (from this lab):

```
ossec.conf:76:    <directories realtime="yes" report_changes="yes">C:\Users\Clientew10\Desktop</directories>

2026/11/12 15:50:40 wazuh-agent: INFO: (6003): Monitoring path: 'c:\users\clientew10\desktop',
with options 'size | permissions | owner | group | mtime | inode | hash_md5 | hash_sha1
| hash_sha256 | attributes | report_changes | realtime'.
```

The options list must include **`realtime`** and **`report_changes`**.

> Other matches like `Ignore 'file' entry '...\Startup\desktop.ini'` are part of
> the default config and unrelated. They only match the word "desktop".

## 3. Test

PowerShell on win-131. Wait about 10 seconds between commands so each one is a separate event:

```powershell
# CREATE  -> "File added to the system."
Set-Content "C:\Users\Clientew10\Desktop\fim-test.txt" "hello wazuh"

# MODIFY  -> "Integrity checksum changed." (+ syscheck.diff)
Add-Content "C:\Users\Clientew10\Desktop\fim-test.txt" "this line was added"

# DELETE  -> "File deleted."
Remove-Item "C:\Users\Clientew10\Desktop\fim-test.txt"
```

## 4. Check the alerts

Dashboard `https://100.100.100.130` → **☰ → File Integrity Monitoring** → agent
**win-131** → **Events**. Expanding the *modified* event shows the
`syscheck.diff` field with the added line.

## Results

From the FIM report exported from the dashboard (agent `001 win-131`,
Windows 10 Enterprise LTSC 2019, Wazuh v4.14.10, manager `alejo99`):

| Path | Description | Count |
|------|-------------|-------|
| `c:\users\clientew10\desktop\fim-test.txt` | File added to the system. | 1 |
| `c:\users\clientew10\desktop\fim-test.txt` | Integrity checksum changed. | 1 |
| `c:\users\clientew10\desktop\fim-test.txt` | File deleted. | 1 |
| `c:\users\clientew10\desktop\ossec.conf` | File added to the system. | 1 |

- Modified: `2026-11-13T03:10:17Z`. Deleted: `2026-11-13T03:10:31Z`.
  Both were detected **in real time**, hours before the next scheduled scan.
- Last full FIM scan: `2026-11-12T20:50:40Z` → `20:50:59Z` (15:50 local time on
  the agent, which is UTC-5, matching the `ossec.log` line above).
- The extra `ossec.conf` alert is a copy of the agent config saved to the Desktop.
  FIM catches *any* new file, which is exactly what it's for.

✅ All three actions (create, modify, delete) raised the expected alerts.

## Getting the report off the Windows VM

The dashboard report downloads as a PDF on the Windows VM. To copy it to the
Ubuntu server, use the OpenSSH client that comes with Windows 10:

```powershell
scp "C:\Users\Clientew10\Downloads\wazuh-module-agents-001-fim-<id>.pdf" priest67@100.100.100.130:/home/priest67/projects/wazuh/FIM/evidence/
```

The first time, answer `yes` to the host-key prompt, then enter the Ubuntu password.
To read the PDF from the Ubuntu terminal:
`sudo apt install -y poppler-utils && pdftotext -layout report.pdf -`.
