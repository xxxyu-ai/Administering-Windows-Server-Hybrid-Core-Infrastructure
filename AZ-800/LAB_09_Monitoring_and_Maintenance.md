# AZ-800: Administering Windows Server Hybrid Core Infrastructure

## Lab9: Implementing storage solutions in Windows Server

This lab focused on implementing Data Deduplication, configuring iSCSI storage, creating Storage Spaces, and testing Storage Spaces Direct.

---

## Lab exercise 1: Implementing Data Deduplication

### Task 1: Install the Data Deduplication role service
In this task, I installed the Data Deduplication role service on SEA-SVR3 and prepared the M: volume for deduplication testing.

#### 1. Role Installation
- **Target Server:** `SEA-SVR3`
- **Role Service:** `Data Deduplication`

#### 2. Volume Preparation
```powershell
Get-Disk
Initialize-Disk -Number 1
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter M
Format-Volume -DriveLetter M -FileSystem ReFS
```

#### 3. Lab File Access
```powershell
New-PSDrive -Name 'X' -PSProvider FileSystem -Root '\\SEA-ADM1\Labfiles'
New-Item -Type Directory -Path 'M:\Data' -Force
Copy-Item -Path X:\Lab09\CreateLabFiles.cmd -Destination M:\Data\ -PassThru
Start-Process -FilePath M:\Data\CreateLabFiles.cmd -PassThru
Set-Location -Path M:\Data
Get-ChildItem -Path .
Get-PSDrive -Name M
```

#### 4. Validation
- Confirmed that the M: volume was created and formatted with ReFS.
- Created sample files for deduplication testing.
- Recorded the initial free space on the M: volume before deduplication.

---

### Task 2: Enable and configure Data Deduplication
In this task, I enabled Data Deduplication on the M: volume on SEA-SVR3.

#### 1. Deduplication Settings
- **Volume:** `M:`
- **Server:** `SEA-SVR3`
- **Optimization Mode:** `General purpose file server`
- **Deduplicate files older than:** `0 days`
- **Throughput Optimization:** `Enabled`

#### 2. Validation
- Opened Server Manager and reviewed disks on SEA-SVR3.
- Enabled Data Deduplication on the M: volume.
- Confirmed the selected optimization settings.

---

### Task 3: Test Data Deduplication
In this task, I triggered a deduplication job and verified the effect on available volume space.

#### 1. Windows Admin Center Setup
- Installed Windows Admin Center if needed.
- Connected to `sea-svr3.contoso.com`.

#### 2. Start Deduplication Job
```powershell
Start-DedupJob -Volume M: -Type Optimization -Memory 50
```

#### 3. Status Checks
```powershell
Get-PSDrive -Name M
Get-DedupStatus -Volume M: | fl
Get-DedupVolume -Volume M: | fl
Get-DedupMetadata -Volume M: | fl
```

#### 4. Validation
- Verified that available free space on M: changed after deduplication.
- Waited for the job to complete and checked the volume again.
- Confirmed deduplication was active on the M: volume.
- Reviewed deduplication rate and savings in the volume properties.

---

### Evidence
> **[Picture]**
> *Capture the ReFS volume creation, sample file creation, deduplication job output, and the deduplication savings shown in Server Manager.*

---

### Professional Insight
- **Data Deduplication:** This feature reduces storage consumption by eliminating duplicate data blocks.
- **ReFS Volume:** Using ReFS is a common requirement for deduplication testing in lab scenarios.
- **Operational Visibility:** Windows Admin Center and Server Manager provide helpful views for monitoring deduplication status and savings.

---

## Lab exercise 2: Configuring iSCSI storage

### Task 1: Install iSCSI and configure targets
In this task, I installed the iSCSI Target Server role on SEA-SVR3 and prepared two ReFS volumes to host iSCSI virtual disks.

#### 1. iSCSI Target Installation
- **Target Server:** `SEA-SVR3`
- **Role Service:** `FS-iSCSITarget-Server`

#### 2. Disk Preparation
```powershell
Install-WindowsFeature -Name FS-iSCSITarget-Server -IncludeManagementTools

Initialize-Disk -Number 2
$partition2 = New-Partition -DiskNumber 2 -UseMaximumSize -AssignDriveLetter
Format-Volume -DriveLetter $partition2.DriveLetter -FileSystem ReFS

Initialize-Disk -Number 3
$partition3 = New-Partition -DiskNumber 3 -UseMaximumSize -AssignDriveLetter
Format-Volume -DriveLetter $partition3.DriveLetter -FileSystem ReFS
```

#### 3. Firewall Configuration
```powershell
New-NetFirewallRule -DisplayName "iSCSITargetIn" -Profile "Any" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 3260
New-NetFirewallRule -DisplayName "iSCSITargetOut" -Profile "Any" -Direction Outbound -Action Allow -Protocol TCP -LocalPort 3260
```

#### 4. Validation
- Confirmed the iSCSI Target Server role was installed.
- Created and formatted two ReFS volumes on disks 2 and 3.
- Added firewall rules to allow iSCSI traffic on TCP 3260.
- Recorded the assigned drive letters for the new volumes.

---

### Task 2: Connect to and configure iSCSI targets
In this task, I created two iSCSI virtual disks on SEA-SVR3 and connected to them from SEA-DC1.

#### 1. iSCSI Virtual Disk Configuration
- **Disk 1 Name:** `iSCSIDisk1`
- **Disk 1 Size:** `5 GB`
- **Disk 1 Type:** `Dynamically Expanding`
- **Target Name:** `iSCSIFarm`
- **Access Server:** `SEA-DC1`

- **Disk 2 Name:** `iSCSIDisk2`
- **Disk 2 Size:** `5 GB`
- **Disk 2 Type:** `Dynamically Expanding`
- **Target:** `iSCSIFarm`

#### 2. Initiator Configuration
- **Initiator Server:** `SEA-DC1`
- **Initiator Service:** `msiscsi`

```powershell
Start-Service msiscsi
iscsicpl
```

#### 3. Validation
- Created the iSCSI virtual disks successfully.
- Connected the initiator on SEA-DC1 to the iSCSI target.
- Verified the target was accessible from the initiator host.

---

### Task 3: Verify iSCSI disk configuration
In this task, I verified that the iSCSI disks appeared on SEA-DC1 and then initialized them for use.

#### 1. Disk Verification
```powershell
Get-Disk
```
- Confirmed that the iSCSI disks were present and initially offline.

#### 2. Volume Creation
```powershell
Initialize-Disk -Number 1
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter E
Format-Volume -DriveLetter E -FileSystem ReFS

Initialize-Disk -Number 2
New-Partition -DiskNumber 2 -UseMaximumSize -DriveLetter F
Format-Volume -DriveLetter F -FileSystem ReFS
```

#### 3. Validation
- Verified that both iSCSI disks were detected on SEA-DC1.
- Initialized and formatted the disks with ReFS.
- Confirmed that both drives became online after formatting.

---

### Task 4: Revert disk configuration
In this task, I reset the disks on SEA-SVR3 to their original state to prepare for the next exercise.

#### 1. Reset Commands
```powershell
for ($num = 1; $num -le 4; $num++) { Clear-Disk -Number $num -RemoveData -RemoveOEM -ErrorAction SilentlyContinue }
for ($num = 1; $num -le 4; $num++) { Set-Disk -Number $num -IsOffline $true }
```

#### 2. Validation
- Confirmed that the target disks were cleared.
- Set the disks back to offline.
- Restored SEA-SVR3 to the required lab state.

---

### Evidence
> **[Picture]**
> *Capture the iSCSI Target Server installation, the created virtual disks, the initiator connection on SEA-DC1, the disk initialization on SEA-DC1, and the disk reset commands on SEA-SVR3.*

---

### Professional Insight
- **Centralized Storage:** iSCSI provides a network-based storage approach that can simplify storage administration.
- **Target and Initiator Roles:** Separating the storage target and the initiator helps model enterprise storage deployment.
- **Lab Reset:** Reverting disk configuration is important to leave the environment ready for subsequent exercises.

---

## Lab exercise 3: Configuring redundant Storage Spaces

### Task 1: Create a storage pool
In this task, I created a new storage pool named SP1 on SEA-SVR3 and brought the required disks online.

#### 1. Storage Pool Configuration
- **Storage Pool Name:** `SP1`
- **Target Server:** `SEA-SVR3`
- **Disk Size:** `127 GB` disks

#### 2. Validation
- Refreshed the Disks pane in Server Manager.
- Set disks 1-4 of SEA-SVR3 to Online.
- Created the storage pool using the available 127 GB disks.

---

### Task 2: Create a volume based on a three-way mirrored disk
In this task, I created a three-way mirrored virtual disk and formatted it as a ReFS volume.

#### 1. Virtual Disk Configuration
- **Virtual Disk Name:** `Three-Mirror`
- **Storage Pool:** `SP1`
- **Layout:** `Mirror`
- **Provisioning:** `Thin`
- **Size:** `25 GB`

#### 2. Volume Configuration
- **Volume Label:** `TestData`
- **File System:** `ReFS`
- **Drive Letter:** `T`

#### 3. Validation
- Created the virtual disk successfully.
- Created and formatted the ReFS volume.
- Confirmed the volume was available as drive `T:`.

---

### Task 3: Manage a volume in File Explorer
In this task, I connected to SEA-SVR3 and created test content on the shared volume.

#### 1. Firewall and File Access
```powershell
Enable-NetFirewallRule -Group "@FirewallAPI.dll,-28502"
```

#### 2. File Creation
- Opened File Explorer.
- Browsed to `\\SEA-SVR3.contoso.com\t$`.
- Created a folder named `TestData`.
- Created a text file named `TestDocument.txt`.

#### 3. Validation
- Verified that the file and folder were created successfully.
- Confirmed the volume was accessible over the network.

---

### Task 4: Disconnect a disk from the storage pool and verify volume availability
In this task, I added the remaining available disk to the pool and removed one of the original disks to test resiliency.

#### 1. Pool Change
- Added the remaining available disk to `SP1` using automatic allocation.
- Removed one of the first three disks from the pool.

#### 2. Validation
- Opened File Explorer and confirmed `TestDocument.txt` was still available.
- Verified that the storage space remained accessible after disk removal.

---

### Task 5: Add a disk to the storage pool and verify volume availability
In this task, I re-scanned the storage pool and returned the removed disk to the pool.

#### 1. Pool Recovery
- Re-scanned the `SP1` storage pool.
- Added the removed disk back with automatic allocation.

#### 2. Validation
- Confirmed `TestDocument.txt` was still available after the disk was added back.
- Verified the storage pool returned to a healthy state.

---

### Task 6: Revert disk configuration
In this task, I removed the virtual disk and storage pool, then reset the disks to their original state.

#### 1. Cleanup Commands
```powershell
Get-VirtualDisk -FriendlyName 'Three-Mirror' | Remove-VirtualDisk
Get-StoragePool -FriendlyName 'SP1' | Remove-StoragePool
for ($num = 1; $num -le 4; $num++) { Clear-Disk -Number $num -RemoveData -RemoveOEM -ErrorAction SilentlyContinue }
for ($num = 1; $num -le 4; $num++) { Set-Disk -Number $num -IsOffline $true }
```

#### 2. Validation
- Removed the virtual disk.
- Removed the storage pool.
- Reset the disks to their original offline state.

---

### Evidence
> **[Picture]**
> *Capture the storage pool creation, the three-way mirror virtual disk, the file creation in File Explorer, the disk removal and re-addition, and the cleanup commands.*

---

### Professional Insight
- **Storage Pools:** Storage pools abstract physical disks into a manageable storage layer.
- **Three-Way Mirror:** This resiliency option helps maintain availability even when disks are removed or fail.
- **Operational Testing:** Removing and re-adding disks is a useful way to validate that the storage space continues to provide access to data.

---

## Lab exercise 4: Implementing Storage Spaces Direct

### Task 1: Prepare for installation of Storage Spaces Direct
In this task, I prepared SEA-SVR1, SEA-SVR2, and SEA-SVR3 for clustering and Storage Spaces Direct.

#### 1. Server Manager Validation
- Verified that `SEA-SVR1`, `SEA-SVR2`, and `SEA-SVR3` showed `Online – Performance counters not started`.
- Opened the Disks pane for File and Storage Services.
- Confirmed disks 1 through 4 on `SEA-SVR3` were listed as `Unknown`.
- Brought all attached disks online on `SEA-SVR1`, `SEA-SVR2`, and `SEA-SVR3`.

#### 2. PowerShell ISE Preparation
- Opened `C:\Labfiles\Lab09\Implement-StorageSpacesDirect.ps1` in Windows PowerShell ISE.
- Reviewed the numbered script steps before execution.

#### 3. Step 1 Commands
- Installed the File Server role and Failover Clustering feature on the three servers.
- Restarted `SEA-SVR1`, `SEA-SVR2`, and `SEA-SVR3`.
- Installed the Failover Cluster Manager tool on `SEA-ADM1`.

#### 4. Validation
- Confirmed role and feature installation completed successfully.
- Verified that the servers restarted as expected.
- Confirmed Failover Cluster Manager was installed on SEA-ADM1.

---

### Task 2: Create and validate the failover cluster
In this task, I validated the cluster configuration and created the failover cluster.

#### 1. Validation
- Opened Failover Cluster Manager.
- Ran the cluster validation test from PowerShell ISE.
- Confirmed that no tests failed.
- Ignored warnings as instructed.

#### 2. Cluster Creation
- Ran the step 3 command to create the cluster.
- **Cluster Name:** `S2DCluster.Contoso.com`

#### 3. Validation
- Added the new cluster to Failover Cluster Manager.
- Confirmed that the cluster was created successfully.

---

### Task 3: Enable Storage Spaces Direct
In this task, I enabled Storage Spaces Direct and created the default cluster storage pool and virtual disks.

#### 1. S2D Enablement
- Ran the step 4 command to enable Storage Spaces Direct on the cluster.

#### 2. Storage Pool
- Ran the step 5 command to create the storage pool.
- **Storage Pool Name:** `S2DStoragePool`

#### 3. Virtual Disks
- Ran the step 6 command to create the virtual disks.

#### 4. Validation
- Confirmed the cluster contained `Cluster Pool 1`.
- Verified that the `Cluster Virtual Disk (CSV)` object appeared in the Disks pane.
- Confirmed the storage pool FriendlyName was `S2DStoragePool`.

---

### Task 4: Create a storage pool, a virtual disk, and a share
In this task, I created the SOFS role and the `VM01` SMB share.

#### 1. SOFS Role
- Ran the step 7 command to create the `S2D-SOFS` role.

#### 2. Share Creation
- Ran all three commands in step 8 to create the `VM01` share.

#### 3. Validation
- Confirmed that `S2D-SOFS` appeared in the Roles pane.
- Confirmed that `VM01` appeared in the Shares pane.

---

### Task 5: Verify Storage Spaces Direct functionality
In this task, I validated cluster resilience by taking one node offline and then restoring it.

#### 1. Share Validation
- Opened `\\s2d-sofs\VM01` in File Explorer.
- Created the folder `VMFolder`.

#### 2. Simulated Failure
```powershell
Stop-Computer -ComputerName SEA-SVR3 -Force
```

#### 3. Verification During Failure
- Refreshed Server Manager and confirmed `SEA-SVR3` was no longer accessible.
- Reviewed the Cluster Virtual Disk (CSV) in Failover Cluster Manager.
- Confirmed the health status changed to `Warning`.
- Confirmed the operational status changed to `Degraded` or `Incomplete`.

#### 4. Windows Admin Center Review
- Added the `S2DCluster.Contoso.com` cluster to Windows Admin Center.
- Verified the alert showing `SEA-SVR3` was not reachable.

#### 5. Recovery
- Restarted `SEA-SVR3`.
- Waited for the alert to clear automatically.
- Refreshed Windows Admin Center and confirmed all servers became healthy again.

---

### Evidence
> **[Picture]**
> *Capture the cluster validation, cluster creation, S2D enablement, SOFS role and share creation, the failure test, and the recovery status in Windows Admin Center.*

---

### Professional Insight
- **Storage Spaces Direct:** S2D turns local disks into a highly available storage platform.
- **Cluster Validation:** Running validation first is essential before creating a production cluster.
- **Resiliency Testing:** Taking one node offline is a useful way to verify that cluster storage remains available during failure.
