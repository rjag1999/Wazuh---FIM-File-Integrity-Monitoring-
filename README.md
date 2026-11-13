# Wazuh: FIM (File Integrity Monitoring)

Hands-on lab of Wazuh **File Integrity Monitoring** (the syscheck module) on a
Windows 10 endpoint.

Part of my Wazuh home lab: Wazuh 4.14.10 all-in-one on Ubuntu Server 24.04
(`100.100.100.130`), with the Windows 10 agent `win-131` (`100.100.100.131`) on the
VMware NAT network. Base setup (network, server install, agent enrollment,
TLS cert) is in my main [Wazuh](https://github.com/rjag1999/Wazuh) repository.

## Exercises

| # | Exercise | Status |
|---|----------|--------|
| 5 | [Desktop folder: realtime + report_changes](docs/05-file-integrity-monitoring.md) | ✅ Done |
| 5b | [Real-world targets: hosts file + Run registry key with who-data](docs/05b-fim-hosts-registry-whodata.md) | ⏳ In progress |

## Evidence
- [`evidence/05-fim-report.pdf`](evidence/05-fim-report.pdf): FIM report exported
  from the dashboard (`fim-test.txt` added / modified / deleted, all detected in real time).

## Skills shown
- Configuring syscheck `<directories>` with `realtime`, `report_changes` and `whodata`
- Verifying agent config through `ossec.conf` / `ossec.log`
- Simulating attacker activity and checking the alerts in the Wazuh dashboard
- Mapping FIM to real-world use (persistence, hosts hijacking, PCI DSS 11.5)
