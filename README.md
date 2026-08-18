# AD Phase 1 — Local Host Enumeration

Command and script reference for **Phase 1** of an internal Active Directory red team engagement: local host enumeration following an initial domain-joined foothold, prior to lateral movement or domain-level attacks.

## Scope

This reference covers three sub-phases:

1. **System & Configuration** — OS/patch level, local admin/service accounts, scheduled tasks, AV/EDR fingerprinting, LAPS, AppLocker/WDAC, sensitive file discovery
2. **Credential Harvesting on Host** — LSASS protections and dumping, SAM/SYSTEM hive extraction, DPAPI secrets, browser/Credential Manager/Wi-Fi/saved-session credentials
3. **Local Privilege Escalation** — automated enumeration (winPEAS/Seatbelt/PowerUp), token privilege abuse, PATH/service/registry misconfigurations, kernel exploit applicability

## Shell tags

Every command in `Phase1_Commands_Scripts.docx` is tagged so there's no ambiguity about where to run it:

| Tag | Meaning |
|---|---|
| 🟩 **CMD or POWERSHELL** | Native command/binary — runs identically in `cmd.exe` and PowerShell |
| 🟦 **POWERSHELL ONLY** | Cmdlet or PS-only syntax (`Get-*`, `$env:`, `[Type]::Method`) — fails in plain `cmd.exe` |
| 🟫 **OFFLINE / ATTACKER MACHINE** | Not run on the target — run against exfiltrated files/hashes on your own box |

If you're dropped into `cmd.exe` and need a PowerShell-only command:
