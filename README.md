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

## Contents

- `Phase1_Commands_Scripts.docx` — full tagged command reference
- `Phase1_Test_Cases.md` *(add if/when generated)* — pass/fail test case table with risk ratings for reporting

## Usage

Each command block maps to a corresponding checklist item and test case. Intended workflow:

1. Run automated sweep (winPEAS / Seatbelt) for a broad first pass
2. Manually verify flagged items using the targeted commands
3. Record result (secure / finding), evidence, and risk rating against the test case table
4. Carry confirmed findings into the engagement report, mapped to MITRE ATT&CK

## ⚠️ Scope & Authorization

This material is intended **strictly for use in authorized penetration testing and red team engagements** covered by a signed contract and Rules of Engagement (RoE). It is not intended for use against systems you do not have explicit written authorization to test.

This repository does not contain any client-identifying information, target hostnames, credentials, or engagement findings. All content here is generic technique/command reference only.

## License

No license granted. This content is for personal/professional reference in authorized engagements — not licensed for redistribution or reuse.
