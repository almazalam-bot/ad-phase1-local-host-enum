# AD Red Team Engagement — Command & Script Reference

Command and script reference for an internal Active Directory red team engagement, starting from an initial domain-joined foothold.

## Phases covered

| Phase | Folder / File | Access assumed |
|---|---|---|
| **Phase 1** — Local Host Enumeration | `Phase1_Commands.md`, `scripts/powershell/1.*`, `scripts/cmd/1.*` | PowerShell + cmd.exe |
| **Phase 2** — Network & Host Discovery | `Phase2_Commands.md`, `scripts/cmd/2_network_discovery.cmd` | **cmd.exe only** — no PowerShell, no pre-existing pivot tooling |

## Repository structure

```
.
├── Phase1_Commands.md
├── Phase2_Commands.md
├── docs/
│   ├── Phase1_Commands_Scripts.docx
│   └── Phase2_Commands_Scripts.docx
└── scripts/
    ├── powershell/
    │   ├── 1.1_system_config.ps1
    │   ├── 1.2_credential_harvesting.ps1
    │   └── 1.3_local_privesc.ps1
    └── cmd/
        ├── 1.1_system_config.cmd
        ├── 1.2_credential_harvesting.cmd
        ├── 1.3_local_privesc.cmd
        └── 2_network_discovery.cmd
```

## Shell / capability tags

**Phase 1** (`Phase1_Commands.md`):

| Tag | Meaning |
|---|---|
| 🟩 CMD or POWERSHELL | Native command — runs identically in cmd.exe and PowerShell |
| 🟦 POWERSHELL ONLY | Cmdlet/PS-only syntax — fails in plain cmd.exe |
| 🟫 OFFLINE / ATTACKER MACHINE | Run on your own box against exfiltrated files/hashes |

**Phase 2** (`Phase2_Commands.md`) — written for cmd.exe-only access:

| Tag | Meaning |
|---|---|
| 🟩 NATIVE CMD | Built into Windows — zero extra tools |
| 🟨 EXTERNAL BINARY REQUIRED | Needs a file transferred onto the target host |
| 🟥 REQUIRES ADMIN / CHANGES HOST STATE | Local admin needed and/or alters host config |
| ⬛ LIMITATION | Not achievable from cmd.exe alone — gap noted explicitly |

## Usage

1. Run automated sweep tools first where available (winPEAS/Seatbelt for Phase 1)
2. Run the phase scripts for targeted/manual verification
3. Record results, evidence, and risk rating against your test case tracker
4. Carry confirmed findings into the engagement report, mapped to MITRE ATT&CK

From cmd.exe, drop into PowerShell if available:
```
powershell -ep bypass -File .\1.1_system_config.ps1
```

## ⚠️ Scope & Authorization

This material is intended **strictly for use in authorized penetration testing and red team engagements** covered by a signed contract and Rules of Engagement (RoE). It is not intended for use against systems you do not have explicit written authorization to test.

This repository does not contain any client-identifying information, target hostnames, credentials, or engagement findings. All content here is generic technique/command reference only.

## License

No license granted. For personal/professional reference in authorized engagements only — not licensed for redistribution or reuse.
