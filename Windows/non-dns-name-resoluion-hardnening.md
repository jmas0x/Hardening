# Non-DNS Name Resolution Hardening 🛡️

This guide shows how to disable alternative name resolution methods (LLMNR, mDNS, NetBIOS) in Windows to prevent man-in-the-middle attacks.

---

## 🔍 Non-DNS Name Resolution Methods (Enabled by Default)

- **LLMNR (Link-Local Multicast Name Resolution)**  
  - Introduced in Windows Vista / Server 2008+.  
  - Port UDP/5355.  
- **mDNS (Multicast DNS)**  
  - Introduced in Windows Server 2019/2022+.  
  - Port UDP/5353.  
- **NetBIOS over TCP/IP**  
  - Legacy protocol.  
  - Ports UDP/137 and TCP/139.  

> ⚠️ These methods respond to short names if DNS fails, exposing devices to man-in-the-middle attacks.

---

## 🧩 Validation Checks

### LLMNR
```powershell
reg query "HKLM\Software\Policies\Microsoft\Windows NT\DNSClient" | FIND "EnableMulticast"
netstat -nao | FIND ":5355"
```

### mDNS
```powershell
reg query "HKLM\System\CurrentControlSet\Services\DNScache\Parameters" | FIND "EnableMDNS"
netstat -nao | FIND ":5353"
```

### NetBIOS
```powershell
ipconfig /all | FIND "netbios"
netstat -nao | FIND ":137"
```

---

## 🛠️ Recommended Disabling Steps

### 🔧 LLMNR
- **Via GPO**:  
  `Computer Configuration → Administrative Templates → Network → DNS Client → Turn off multicast name resolution = ENABLED`
- **Via Registry**:
```powershell
REG ADD "HKLM\Software\Policies\Microsoft\Windows NT\DNSClient" /v EnableMulticast /t REG_DWORD /d 0 /f
```

### 🔧 mDNS
- **Via Group Policy Preference (GPP)**:
```plaintext
Hive: HKEY_LOCAL_MACHINE
Path: System\CurrentControlSet\Services\DNScache\Parameters
Name: EnableMDNS
Type: REG_DWORD
Data: 0
```

### 🔧 NetBIOS
- **Via DHCP**: Set Option 001 to value `0x2` at the DHCP scope.  
- **Via GUI**: Network Interface → IPv4 → Advanced → WINS → Disable NetBIOS over TCP/IP.  
- **Via Registry**:
```powershell
# For each NIC (GUID)
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces\tcpip_{GUID}" -Name NetbiosOptions -Value 2 -PropertyType DWORD -Force
```

---

## ✅ Quick Summary

| Protocol   | Validation                | Disabling Method           |
|------------|---------------------------|----------------------------|
| **LLMNR**  | `netstat :5355` / registry | GPO or registry            |
| **mDNS**   | `netstat :5353` / registry | GPP (registry)             |
| **NetBIOS**| `netstat :137` / ipconfig  | DHCP, GUI, or registry     |

---

## 📚 References

- Rickard Nobel — [Disable the non‑DNS name resolution methods in Windows](https://rickardnobel.se/disable-the-non-dns-name-resolution-methods-in-windows/) (2023)  
- Susan Bradley, CSO Online — “How to disable LLMNR in Windows Server” (2019)  
- Netwrix — Recommendations for disabling LLMNR via GPO  

---
