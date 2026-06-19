### Watch me do it here: https://www.loom.com/share/594defb3d5a94791b5c23279c31a8a62

---

# SOP: Patching TLS Vulnerabilities with Nessus Essentials


**Document ID:** SOP-SEC-TLS-001  
**Version:** 1.0  
**Classification:** Internal Use Only  
**Owner:** Cybersecurity Operations  
**Last Reviewed:** June 2026  
**Applies To:** Azure Virtual Machines (Windows Server)  
**Tools Required:** Nessus Essentials, PowerShell, Azure Portal  
**CVSS Score (Ref):** 6.5 (Medium) — TLS 1.0/1.1 Enabled  

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Scope](#2-scope)
3. [Prerequisites](#3-prerequisites)
4. [Procedure](#4-procedure)
   - [Phase 1: Run Initial Nessus Scan](#phase-1-run-initial-nessus-vulnerability-scan)
   - [Phase 2: Connect to the Target VM](#phase-2-connect-to-the-target-vm)
   - [Phase 3: Apply the TLS Remediation Script](#phase-3-apply-the-tls-remediation-script)
   - [Phase 4: Restart the VM](#phase-4-restart-the-virtual-machine)
   - [Phase 5: Verify with Nessus Rescan](#phase-5-verify-remediation-with-nessus-rescan)
5. [Troubleshooting](#5-troubleshooting)
6. [Documentation & Change Control](#6-documentation--change-control)
7. [Registry Key Reference](#7-registry-key-reference)
8. [Related Resources](#8-related-resources)
9. [Revision History](#9-revision-history)

---

## 1. Purpose

This SOP describes the process for identifying, remediating, and verifying Transport Layer Security (TLS) vulnerabilities on Azure virtual machines using Nessus Essentials. Specifically, it covers disabling the deprecated TLS 1.0 and TLS 1.1 protocols and enforcing TLS 1.2 and TLS 1.3 on Windows Server systems.

> **Why this matters:** TLS 1.0 and TLS 1.1 are cryptographically weak and have known vulnerabilities (POODLE, BEAST). NIST SP 800-52 Rev. 2 and compliance frameworks (PCI-DSS, FedRAMP) require TLS 1.2+ only. Leaving deprecated versions enabled produces a CVSS Medium finding (6.5) on vulnerability scans.

---

## 2. Scope

**In scope:**
- Windows Server VMs hosted in Microsoft Azure
- Systems where a Nessus scan reveals TLS 1.0 or TLS 1.1 as enabled
- IT Support Technicians, System Administrators, and Security Operations personnel performing vulnerability remediation

**Out of scope:**
- Linux-based VMs (separate mechanism required)
- Application-layer TLS configuration (IIS, Apache, Nginx — see application-specific SOPs)
- Nessus installation, licensing, or credentialed scan setup

---

## 3. Prerequisites

### 3.1 Access Requirements

- Azure Portal access (Contributor role minimum) on the target VM
- RDP or Azure Bastion access to the Windows Server VM
- Local Administrator or Domain Administrator credentials on the VM
- Nessus Essentials installed on a scan host with network reachability to the target VM

### 3.2 Knowledge Requirements

- Basic familiarity with Nessus scan configuration and report review
- Basic PowerShell usage on Windows Server
- Understanding of Windows Registry structure

### 3.3 Tools

| Tool | Version / Edition | Purpose |
|------|-------------------|---------|
| Nessus Essentials | Latest (free tier) | Vulnerability scanning |
| PowerShell | 5.1+ (built-in) | Registry patching via script |
| Azure Portal | portal.azure.com | VM management & restart |
| RDP / Azure Bastion | Built-in / Azure Bastion | Remote access to VM |

---

## 4. Procedure

### Phase 1: Run Initial Nessus Vulnerability Scan

<img width="2347" height="1004" alt="image" src="https://github.com/user-attachments/assets/dd9e3d6f-4068-4589-b66a-4bbb8f239ad1" />
<img width="1715" height="609" alt="image" src="https://github.com/user-attachments/assets/e3573520-c2b6-45c0-8506-ea1243f1cd68" />



1. Open Nessus Essentials in a browser (default: `https://localhost:8834`) on the scan host.
2. Navigate to **My Scans** and select or create a **Basic Network Scan** targeting the Azure VM IP address.
3. Confirm the scan is configured with valid credentials (credentialed scan recommended for accurate results).
4. Launch the scan and wait for it to complete.
5. Open the scan results and search for `TLS` in the plugin search bar.
6. Identify the **TLS 1.0 Enabled** and/or **TLS 1.1 Enabled** findings. Note the CVSS score and Plugin ID for your change record.

> **Note:** The expected Nessus findings are:
> - Plugin #104743 — *TLS Version 1.0 Protocol Detection*
> - Plugin #157288 — *TLS Version 1.1 Protocol Deprecated*
>
> Screenshot and record these findings before remediation.

---

### Phase 2: Connect to the Target VM

1. Log in to the Azure Portal (`portal.azure.com`).
2. Navigate to **Virtual Machines** and select the target VM.
3. Verify the VM is in a **Running** state.
4. Connect via **RDP** or **Azure Bastion** using Administrator credentials.
5. Once logged in, open **PowerShell as Administrator** (right-click Start → Windows PowerShell (Admin)).

---

### Phase 3: Apply the TLS Remediation Script

Copy and paste the full script below into an elevated PowerShell session. This script disables TLS 1.0 and TLS 1.1 and explicitly enables TLS 1.2 and TLS 1.3 via the Windows Registry.

> **Critical:** Both `Client` and `Server` registry keys must be set for each protocol. Setting only the `Server` key is a common mistake that results in partial remediation and will **not** clear the Nessus finding on rescan.

```powershell
# TLS Remediation Script v2 - Disable TLS 1.0 & 1.1, Enable TLS 1.2 & 1.3
# Run as Local Administrator in PowerShell

$tlsBase = 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols'

# -- Disable TLS 1.0 ----------------------------------------------------------
New-Item -Path "$tlsBase\TLS 1.0\Client" -Force | Out-Null
New-Item -Path "$tlsBase\TLS 1.0\Server" -Force | Out-Null
Set-ItemProperty -Path "$tlsBase\TLS 1.0\Client" -Name 'Enabled'           -Value 0 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.0\Client" -Name 'DisabledByDefault' -Value 1 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.0\Server" -Name 'Enabled'           -Value 0 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.0\Server" -Name 'DisabledByDefault' -Value 1 -Type DWord

# -- Disable TLS 1.1 ----------------------------------------------------------
New-Item -Path "$tlsBase\TLS 1.1\Client" -Force | Out-Null
New-Item -Path "$tlsBase\TLS 1.1\Server" -Force | Out-Null
Set-ItemProperty -Path "$tlsBase\TLS 1.1\Client" -Name 'Enabled'           -Value 0 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.1\Client" -Name 'DisabledByDefault' -Value 1 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.1\Server" -Name 'Enabled'           -Value 0 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.1\Server" -Name 'DisabledByDefault' -Value 1 -Type DWord

# -- Enable TLS 1.2 -----------------------------------------------------------
New-Item -Path "$tlsBase\TLS 1.2\Client" -Force | Out-Null
New-Item -Path "$tlsBase\TLS 1.2\Server" -Force | Out-Null
Set-ItemProperty -Path "$tlsBase\TLS 1.2\Client" -Name 'Enabled'           -Value 1 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.2\Client" -Name 'DisabledByDefault' -Value 0 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.2\Server" -Name 'Enabled'           -Value 1 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.2\Server" -Name 'DisabledByDefault' -Value 0 -Type DWord

# -- Enable TLS 1.3 -----------------------------------------------------------
New-Item -Path "$tlsBase\TLS 1.3\Client" -Force | Out-Null
New-Item -Path "$tlsBase\TLS 1.3\Server" -Force | Out-Null
Set-ItemProperty -Path "$tlsBase\TLS 1.3\Client" -Name 'Enabled'           -Value 1 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.3\Client" -Name 'DisabledByDefault' -Value 0 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.3\Server" -Name 'Enabled'           -Value 1 -Type DWord
Set-ItemProperty -Path "$tlsBase\TLS 1.3\Server" -Name 'DisabledByDefault' -Value 0 -Type DWord

Write-Host 'TLS remediation complete. Restart the VM to apply changes.' -ForegroundColor Green
```

---

### Phase 4: Restart the Virtual Machine

Registry-based TLS changes are **not applied until the OS is restarted.**

1. Confirm the script completed without errors.
2. Schedule a restart during an approved maintenance window if the VM is production.
3. Restart using one of the following:
   - **From within the VM (PowerShell):** `Restart-Computer -Force`
   - **Azure Portal:** Virtual Machines → select VM → Overview → **Restart**
4. Wait for the VM to come back online and confirm RDP/Bastion connectivity.

---

### Phase 5: Verify Remediation with Nessus Rescan

<img width="1689" height="842" alt="image" src="https://github.com/user-attachments/assets/bc4daaa1-3d81-4ac5-a476-06d711abd205" />
<img width="1717" height="663" alt="image" src="https://github.com/user-attachments/assets/28934f63-1155-4634-8deb-5bdb5846c3d7" />


1. Return to Nessus Essentials on the scan host.
2. Run the same scan configuration against the target VM used in Phase 1.
3. After the scan completes, review results for TLS 1.0 and TLS 1.1 findings.
4. Confirm that **TLS 1.0 Enabled** and **TLS 1.1 Enabled** findings no longer appear.
5. Export the post-remediation scan report (PDF or CSV) for your change record.

> ✅ **Success:** Remediation is confirmed when the TLS 1.0 and TLS 1.1 findings are absent from the rescan report. If they still appear, see the Troubleshooting section below.

---

## 5. Troubleshooting

| Symptom | Likely Cause | Resolution |
|---------|--------------|------------|
| TLS findings still present after rescan | VM not restarted, or only `Server` key was set (not `Client`) | Confirm restart occurred. Re-run the full v2 script ensuring both `Client` and `Server` keys are set for TLS 1.0 and 1.1. |
| Script runs but no output or partial output | Non-elevated PowerShell session | Close PowerShell. Re-open as Administrator and re-run. |
| Applications break after restart | Application hard-coded to TLS 1.0/1.1 | Identify and update the application to support TLS 1.2+. Temporarily re-enable TLS 1.1 only if a critical business need exists — document the exception. |
| Registry keys not visible after script | Incorrect path or insufficient permissions | Manually verify in `regedit.exe` at `HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols`. |

---

## 6. Documentation & Change Control

The following records must be created or updated as part of this remediation:

- [ ] Pre-scan Nessus report (PDF or CSV) showing TLS 1.0/1.1 findings
- [ ] Change ticket documenting: VM name, date/time, technician, script version used, restart time
- [ ] Post-scan Nessus report confirming findings are resolved
- [ ] If application issues occur, document the exception and escalate to the security team for risk acceptance

---

## 7. Registry Key Reference

Expected end state after remediation:

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\
  TLS 1.0\
    Client\  Enabled = 0    DisabledByDefault = 1
    Server\  Enabled = 0    DisabledByDefault = 1
  TLS 1.1\
    Client\  Enabled = 0    DisabledByDefault = 1
    Server\  Enabled = 0    DisabledByDefault = 1
  TLS 1.2\
    Client\  Enabled = 1    DisabledByDefault = 0
    Server\  Enabled = 1    DisabledByDefault = 0
  TLS 1.3\
    Client\  Enabled = 1    DisabledByDefault = 0
    Server\  Enabled = 1    DisabledByDefault = 0
```

> **Note:** TLS 1.3 registry key support varies by OS version. On Windows Server 2019+, TLS 1.3 is enabled by default and may not require explicit registry keys. On Windows Server 2016, TLS 1.3 is not supported at the OS level regardless of registry settings.

---

## 8. Related Resources

- Nessus Plugin #104743 — TLS Version 1.0 Protocol Detection
- Nessus Plugin #157288 — TLS Version 1.1 Protocol Deprecated
- [NIST SP 800-52 Rev. 2 — Guidelines for TLS Implementations](https://csrc.nist.gov/publications/detail/sp/800-52/rev-2/final)
- [Microsoft KB3140245 — Enable TLS 1.1 and 1.2 as default in WinHTTP](https://support.microsoft.com/en-us/topic/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-winhttp-in-windows-c4bd73d2-31d7-761e-0d87-b049e2f452c2)
- [CIS Microsoft Windows Server Benchmark](https://www.cisecurity.org/benchmark/microsoft_windows_server)
- MITRE ATT&CK T1600 — Weaken Encryption (defensive mapping)
