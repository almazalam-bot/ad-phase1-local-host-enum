# AD Phase 1 — Local Host Enumeration

Command and script reference for **Phase 1** of an internal Active Directory red team engagement: local host enumeration following an initial domain-joined foothold, prior to lateral movement or domain-level attacks.

## Scope

This reference covers three sub-phases:

1. **System & Configuration** — OS/patch level, local admin/service accounts, scheduled tasks, AV/EDR fingerprinting, LAPS, AppLocker/WDAC, sensitive file discovery
2. **Credential Harvesting on Host** — LSASS protections and dumping, SAM/SYSTEM hive extraction, DPAPI secrets, browser/Credential Manager/Wi-Fi/saved-session credentials
3. **Local Privilege Escalation** — automated enumeration (winPEAS/Seatbelt/PowerUp), token privilege abuse, PATH/service/registry misconfigurations, kernel exploit applicability

## Repository structure

```
.
├── Phase1_Commands.md              # full command reference, renders on GitHub
├── docs/
│   └── Phase1_Commands_Scripts.docx  # same content as a Word doc
└── scripts/
    ├── powershell/
    │   ├── 1.1_system_config.ps1
    │   ├── 1.2_credential_harvesting.ps1
    │   └── 1.3_local_privesc.ps1
    └── cmd/
        ├── 1.1_system_config.cmd
        ├── 1.2_credential_harvesting.cmd
        └── 1.3_local_privesc.cmd
```

Each phase has **two scripts** — one for native commands (`.cmd`, runs in cmd.exe or PowerShell) and one for PowerShell-only commands (`.ps1`). Run both to cover the full check.

## Shell tags (in Phase1_Commands.md)

| Tag | Meaning |
|---|---|
| 🟩 **CMD or POWERSHELL** | Native command/binary — runs identically in `cmd.exe` and PowerShell |
| 🟦 **POWERSHELL ONLY** | Cmdlet or PS-only syntax — fails in plain `cmd.exe` |
| 🟫 **OFFLINE / ATTACKER MACHINE** | Not run on the target — run against exfiltrated files/hashes on your own box |

If in `cmd.exe` and a PowerShell-only script is needed:
```
powershell -ep bypass -File .\1.1_system_config.ps1
```

## Usage

1. Transfer the relevant `.ps1`/`.cmd` script to the target host (or paste commands manually per the RoE's tooling constraints)
2. Run automated sweep tools (winPEAS/Seatbelt) first for a broad first pass
3. Run the phase scripts for targeted/manual verification
4. Record results, evidence, and risk rating against your test case tracker
5. Carry confirmed findings into the engagement report, mapped to MITRE ATT&CK

## ⚠️ Scope & Authorization

This material is intended **strictly for use in authorized penetration testing and red team engagements** covered by a signed contract and Rules of Engagement (RoE). It is not intended for use against systems you do not have explicit written authorization to test.

This repository does not contain any client-identifying information, target hostnames, credentials, or engagement findings. All content here is generic technique/command reference only.

## License

No license granted. For personal/professional reference in authorized engagements only — not licensed for redistribution or reuse.
