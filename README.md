# Remediation-Email

Hi Team,

Following the completion of our vulnerability assessment, I have prepared the remediation commands and scripts below to address the identified security findings. These remediation actions can be integrated into your endpoint management platform (e.g., Microsoft SCCM or Intune) to streamline deployment.

Please ensure all commands are tested in a non-production environment before deployment to production systems.

**Note:** All commands must be executed with Administrator privileges.

---

### 1. Windows OS Updates (Re-Enable Automatic Updates)

Run the following command in an elevated Command Prompt:

```shell
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v ScheduledInstallDay /t REG_DWORD /d 0 /f; reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 0 /f; reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v AUOptions /t REG_DWORD /d 4 /f
```

After applying the configuration, navigate to **Settings > Windows Update** and install all available updates.

Verification:

```shell
reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"
```

---

### 2. Guest Account Hardening

Remove the Guest account from the local Administrators group and disable the account.

```shell
net localgroup Administrators Guest /delete
```

```shell
net user Guest /active:no
```

---

### 3. Removal of Unsupported Software (Wireshark & WinPcap)

Run the following PowerShell script as Administrator to remove outdated versions of Wireshark and WinPcap:

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/kenbananola/ken-remediation-scripts/main/wireshark-winpcap-force-remove.ps1" -OutFile "$env:TEMP\wireshark-remove.ps1"; PowerShell -ExecutionPolicy Bypass -File "$env:TEMP\wireshark-remove.ps1"
```

Alternatively, the script can be reviewed and downloaded manually before deployment.

---

### 4. SMB Signing Configuration

Enable SMB Signing to reduce the risk of SMB relay and man-in-the-middle attacks.

```powershell
Set-SmbServerConfiguration -RequireSecuritySignature $true -EnableSecuritySignature $true -Force
```

Verification:

```powershell
Get-SmbServerConfiguration | Select RequireSecuritySignature, EnableSecuritySignature
```

Both values should return **True**.

---

### 5. Remote Desktop Network Level Authentication (NLA)

Enable Network Level Authentication for Remote Desktop Services.

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "UserAuthentication" -Value 1 -Force
```

Verification:

```powershell
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "UserAuthentication"
```

The value of **UserAuthentication** should return **1**.

---

### 6. LAN Manager Authentication Hardening

Configure systems to use the recommended LAN Manager compatibility level.

```shell
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v LmCompatibilityLevel /t REG_DWORD /d 5 /f
```

Verification:

```shell
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v LmCompatibilityLevel
```

The value should return **0x5**.

---

Please document any deployment issues, compatibility concerns, or remediation exceptions and report them for review. A follow-up vulnerability scan will be conducted to validate remediation efforts and confirm risk reduction across the environment.

If you have any questions or require assistance with implementation, please let me know.

Best Regards,

**Waleed Alharbi**
Security Analyst
Vulnerability Management & Security Operations
