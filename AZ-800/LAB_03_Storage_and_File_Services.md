# AZ-800: Administering Windows Server Hybrid Core Infrastructure

## Lab3: Managing Windows Server

This lab focused on installing Windows Admin Center, adding servers for remote administration, configuring extensions, verifying remote administration features, and managing a server through Remote PowerShell.

---

## Exercise 1: Implementing and using remote server administration

### Task 1: Install Windows Admin Center
In this task, I downloaded and installed Windows Admin Center on SEA-ADM1 by using PowerShell.

#### 1. Installation
```powershell
$parameters = @{
  Source = "https://aka.ms/WACdownload"
  Destination = ".\WindowsAdminCenter.exe"
}
Start-BitsTransfer @parameters
Start-Process -FilePath '.\WindowsAdminCenter.exe' -ArgumentList '/VERYSILENT' -Wait
```

#### 2. Validation
- Confirmed that Windows Admin Center downloaded successfully.
- Confirmed that the installation completed successfully.
- Restarted SEA-ADM1 if needed to resolve connection issues.

---

### Task 2: Add servers for remote administration
In this task, I connected to Windows Admin Center and added SEA-DC1 as a managed server.

#### 1. Connection Setup
- **Windows Admin Center URL:** `https://SEA-ADM1.contoso.com`
- **Added Server:** `sea-dc1.contoso.com`

#### 2. Validation
- Confirmed the All connections page displayed `sea-adm1.contoso.com`.
- Added `sea-dc1.contoso.com` to the connection list.
- Signed in with the instructor-provided credentials when prompted.

---

### Task 3: Configure Windows Admin Center extensions
In this task, I installed the DNS extension and used Windows Admin Center to manage DNS on SEA-DC1.

#### 1. Extension Installation
- **Extension Installed:** `DNS`

#### 2. DNS Management
- Connected to `sea-dc1.contoso.com`.
- Installed the DNS PowerShell tools.
- Opened the `Contoso.com` zone.
- Reviewed the DNS records in the zone.

#### 3. Validation
- Confirmed that the DNS extension appeared in the installed extensions list.
- Verified DNS management was available through Windows Admin Center.

---

### Task 4: Verify remote administration
In this task, I used Windows Admin Center to verify remote management features on SEA-DC1.

#### 1. Server Management
- Reviewed the Overview pane in Windows Admin Center.
- Confirmed server details and performance information were displayed.

#### 2. Feature Configuration
- Installed `Telnet Client` using Roles & features.
- Enabled `Remote Desktop` from the Settings interface.

#### 3. Remote Desktop Validation
- Connected to `sea-dc1.contoso.com` through Remote Desktop.
- Disconnected from the remote session successfully.

---

### Task 5: Administer servers with Remote PowerShell
In this task, I used PowerShell remoting to manage the Application Identity service on SEA-DC1.

#### 1. Remote Session
```powershell
Enter-PSSession -ComputerName SEA-DC1
```

#### 2. Service Status Check
```powershell
Get-Service -Name AppIDSvc
```
- Confirmed the service was initially stopped.

#### 3. Start the Service
```powershell
Start-Service -Name AppIDSvc
Get-Service -Name AppIDSvc
```
- Confirmed the service was started successfully.
- Verified the service status changed to Running.

---

### Evidence
> **[Picture]**
> *Capture Windows Admin Center showing the added server, DNS extension, DNS zone records, remote desktop session, and PowerShell output for the AppIDSvc service.*

---

### Professional Insight
- **Windows Admin Center:** WAC is useful for browser-based administration of Server Core and other remote Windows servers.
- **Extensions:** Extensions like DNS add management capabilities without requiring full graphical tools on the server.
- **PowerShell Remoting:** Remote PowerShell remains a fast and reliable way to manage services and server settings.
