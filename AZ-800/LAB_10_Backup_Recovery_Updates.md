# AZ-800: Administering Windows Server Hybrid Core Infrastructure

## Lab10: Implementing Azure File Sync

This lab focused on implementing DFS Replication in the on-premises environment as a baseline for migration to Azure File Sync.

---

## Exercise 1: Implementing DFS Replication in your on-premises environment

### Task 1: Deploy DFS
In this task, I installed the DFS management tools and used the provided script to create a DFS namespace and replication group.

#### 1. DFS Management Tools
```powershell
Install-WindowsFeature -Name RSAT-DFS-Mgmt-Con -IncludeManagementTools
```

#### 2. Lab Script
- Opened `C:\Labfiles\Lab10\L10-DeployDFS.ps1` in Windows PowerShell ISE.
- Reviewed the script in the script pane.
- Executed the script to deploy the DFS namespace and replication group.

#### 3. Validation
- Confirmed the DFS namespace and DFS replication group were created successfully.
- Verified that the lab environment was ready for replication testing.

---

### Task 2: Test DFS deployment
In this task, I validated DFS namespace and replication group behavior by comparing replicated content between SEA-SVR1 and SEA-SVR2.

#### 1. DFS Management Review
- Added `\\Contoso.com\Root\` to DFS Management.
- Added the `Branch1` replication group.

#### 2. Validation
- Confirmed `\\Contoso.com\Root\Data` had targets on `SEA-SVR1` and `SEA-SVR2`.
- Verified that the `Branch1` replication group had members `SEA-SVR1` and `SEA-SVR2`.
- Confirmed the replicated folder structure was present on both servers.

#### 3. Replication Test
- Opened File Explorer to `\\SEA-SVR1\Data` and `\\SEA-SVR2\Data`.
- Created a new file with my name in `\\SEA-SVR1\Data`.
- Confirmed that the file replicated to `\\SEA-SVR2\Data` after a few seconds.

#### 4. Validation
- Verified that DFS Replication was functioning correctly.
- Confirmed both File Explorer windows eventually showed the same content.

---

### Professional Insight
- **DFS Replication:** DFS-R is useful for keeping file shares synchronized across servers.
- **Namespace Management:** DFS namespaces provide a consistent path for users regardless of the backing server.
- **Migration Baseline:** Verifying DFS-R behavior first provides a good baseline before moving to Azure File Sync.

---

## Exercise 2: Creating and configuring a sync group

### Task 1: Create an Azure file share
In this task, I created a storage account and an Azure file share that will be used by Azure File Sync.

#### 1. Storage Account
- **Resource Group:** `AZ800-L1001-RG`
- **Redundancy:** `Locally-redundant storage (LRS)`
- **Region:** `<same Azure region used in this lab>`

#### 2. File Share
- **File Share Name:** `share1`

#### 3. Validation
- Created the storage account successfully.
- Created the `share1` Azure file share in the storage account.

---

### Task 2: Use an Azure file share
In this task, I uploaded a file, created a snapshot, and tested restore behavior in the Azure file share.

#### 1. File Upload
- Uploaded `C:\Labfiles\Lab10\File1.txt` to `share1`.

#### 2. Snapshot and Mount
- Created a snapshot of `share1`.
- Mounted `share1` as drive `Z` using the Azure portal connection script.

#### 3. File Modification and Restore
- Opened `File1.txt` from the mounted drive.
- Added my name and saved the file.
- Used **Previous Versions** to restore the earlier version.
- Confirmed that the restored file no longer contained my name.

#### 4. Validation
- Verified that snapshot-based restore worked correctly.
- Confirmed the file share content could be mounted and edited from SEA-ADM1.

---

### Task 3: Deploy Storage Sync Service and a File Sync group
In this task, I created the Azure File Sync resource and a sync group tied to the Azure file share.

#### 1. Storage Sync Service
- **File Sync Resource Name:** `FileSync1`
- **Region:** `<same region used for the storage account>`
- **Resource Group:** `AZ800-L1001-RG`

#### 2. Sync Group
- **Sync Group Name:** `Sync1`
- **Cloud Endpoint:** `share1`
- **Storage Account:** `the storage account created earlier`

#### 3. Validation
- Created the Storage Sync Service successfully.
- Created the `Sync1` sync group.
- Verified that no server was currently registered with `FileSync1`.

---

### Professional Insight
- **Azure File Shares:** Azure file shares provide cloud-based SMB storage that can support hybrid file workflows.
- **Snapshots:** Share snapshots are useful for point-in-time restore and validation.
- **Azure File Sync:** Sync groups create the foundation for hybrid file replication between Azure and on-premises servers.

---

## Exercise 3: Replacing DFS Replication with File Sync-based replication

### Task 1: Add SEA-SVR1 as a server endpoint
In this task, I installed the Azure File Sync agent on SEA-SVR1, registered it with the Storage Sync Service, and added the server endpoint.

#### 1. Agent Installation
- Downloaded `StorageSyncAgent_WS2022.msi` from the Azure portal.
- Saved the installer to `C:\Labfiles\Lab10`.
- Opened `Install-FileSyncServerCore.ps1` in Windows PowerShell ISE.
- Executed the script to install the Azure File Sync agent on `SEA-SVR1`.

#### 2. Registration
- Authenticated to the Azure subscription when prompted.
- Refreshed the registered servers view in `FileSync1`.
- Confirmed that `SEA-SVR1.Contoso.com` was registered.

#### 3. Server Endpoint
- Added `S:\Data` on `SEA-SVR2.Contoso.com` as a server endpoint to `Sync1`.

#### 4. Validation
- Opened `\\SEA-SVR1\Data` and confirmed that `File1.txt` was not yet present there.
- Verified that `File1.txt` was available in the synced data set after endpoint configuration.

---

### Task 2: Register SEA-SVR2 with File Sync
In this task, I modified the installation script and registered SEA-SVR2 with Azure File Sync.

#### 1. Script Update
- Replaced `SEA-SVR1` with `SEA-SVR2` in `Install-FileSyncServerCore.ps1`.
- Saved the updated script.

#### 2. Agent Installation
- Ran `C:\Labfiles\Lab10\Install-FileSyncServerCore.ps1`.
- Authenticated to the Azure subscription when prompted.

#### 3. Validation
- Verified that both `SEA-SVR2.Contoso.com` and `SEA-SVR1.Contoso.com` were registered with `FileSync1`.

---

### Task 3: Remove DFS Replication and add SEA-SVR2 as a server endpoint
In this task, I removed DFS Replication and completed the migration by adding SEA-SVR2 to the File Sync sync group.

#### 1. DFS Removal
- Used DFS Management to delete the `Branch1` replication group.

#### 2. Server Endpoint Addition
- Added `S:\Data` on `SEA-SVR2.Contoso.com` as a server endpoint to `Sync1`.

#### 3. Validation
- Confirmed the DFS Replication relationship was removed.
- Verified that Azure File Sync now handled replication for the file servers.

---

### Professional Insight
- **Migration Strategy:** Azure File Sync can replace DFS Replication while preserving access to synchronized file data.
- **Server Registration:** A server must be registered with the Storage Sync Service before a server endpoint can be created.
- **Endpoint Management:** Moving from DFS-R to File Sync simplifies hybrid replication and cloud integration.

---

## Exercise 4: Verifying replication and enabling cloud tiering

### Task 1: Verify File Sync
In this task, I confirmed that File Sync was replicating content between SEA-SVR1 and SEA-SVR2.

#### 1. Replication Validation
- Opened `\\SEA-SVR1\Data` in File Explorer.
- Opened `\\SEA-SVR2\Data` in a second File Explorer window.
- Created a file with an arbitrary name in `\\SEA-SVR1\Data`.

#### 2. Validation
- Confirmed that the same file appeared in `\\SEA-SVR2\Data` shortly afterward.
- Verified that File Sync had replaced DFS Replication for content replication.

---

### Task 2: Enable cloud tiering
In this task, I enabled cloud tiering for the SEA-SVR1 endpoint and forced tiering to occur immediately.

#### 1. Cloud Tiering Settings
- **Server Endpoint:** `SEA-SVR1.Contoso.com`
- **Free Disk Space Policy:** `80 percent`
- **Date Policy:** Cache files accessed in the last `7 days`

#### 2. File Explorer View
- Added the **Attributes** column to the `\\SEA-SVR1\Data` File Explorer view.

#### 3. Force Tiering
```powershell
Enter-PSSession -ComputerName SEA-SVR2
fsutil file createnew S:\Data\report1.docx 254321098
fsutil file createnew S:\Data\report2.docx 254321098
fsutil file createnew S:\Data\report3.docx 254321098
fsutil file createnew S:\Data\report4.docx 254321098
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncCloudTiering -Path S:\Data
```

#### 4. Validation
- Confirmed cloud tiering was enabled for the server endpoint.
- Verified that tiering could be forced from PowerShell.
- Opened `\\SEA-SVR2\Data` in File Explorer and identified files with attributes such as `L`, `M`, and `O`, indicating tiering had occurred.

---

### Professional Insight
- **Replication Validation:** Confirming that changes appear on both servers proves that Azure File Sync is working correctly.
- **Cloud Tiering:** Tiering helps keep frequently used data local while moving cooler data to Azure.
- **Storage Optimization:** The free-space policy and date policy let you balance local capacity and cloud storage usage.

---

## Exercise 5: Troubleshooting replication issues

### Task 1: Monitor File Sync replication
In this task, I generated sync traffic and reviewed the Azure File Sync monitoring data.

#### 1. Sync Traffic Generation
- Copied the `C:\Windows\INF` folder to `\\SEA-SVR1\Data`.

#### 2. Monitoring Review
- Opened the `Sync1` sync group in the `FileSync1` Storage Sync Service.
- Reviewed the health of both server endpoints.
- Selected the `SEA-SVR1.Contoso.com` endpoint and reviewed **Sync Activity**.
- Examined the **Files Synced** graph and used filters to inspect activity.
- Verified that the `INF` folder was syncing to drive `Z`.

#### 3. Validation
- Confirmed that the `INF` folder generated visible sync traffic.
- Verified the `Files Synced` and `Bytes Synced` graphs reflected the activity.
- Refreshed the Azure portal when needed to see updated statistics.

---

### Task 2: Test replication conflict resolution
In this task, I deliberately created a file conflict to observe how Azure File Sync handled the conflict.

#### 1. Conflict Setup
- Opened `\\SEA-SVR1\Data` and `\\SEA-SVR2\Data` side by side in File Explorer.
- Created `Demo.txt` in both locations.
- Added different text to each file and saved them immediately.

#### 2. Validation
- Confirmed that File Sync detected the conflict.
- Verified that additional conflict files appeared, such as `Demo-SEA-SVR2.txt`.
- Noted that `Demo-Cloud.txt` might also appear depending on the conflict resolution path.
- Reviewed both folders and confirmed that Azure File Sync preserved conflicting versions rather than overwriting them.

---

### Professional Insight
- **Monitoring:** The Azure portal provides useful sync activity graphs for validating replication traffic.
- **Conflict Handling:** Azure File Sync preserves conflicting edits by renaming the conflict copy instead of silently discarding data.
- **Operational Awareness:** Monitoring sync health and conflict files helps identify issues early before they affect users.

---

## Exercise 6: Cleaning up the Azure subscription

### Task 1: Delete the Azure resources that were created in the lab
In this task, I removed the Azure File Sync resources in the required order and deleted the resource group.

#### 1. Cleanup Order
- Removed `SEA-SVR1.Contoso.com` as a registered server.
- Removed `SEA-SVR2.Contoso.com` as a registered server.
- Deleted the `share1` cloud endpoint in the `Sync1` sync group.
- Deleted the `Sync1` sync group.
- Deleted the `FileSync1` Storage Sync Service.
- Deleted the Azure storage account created in the lab.
- Deleted the `AZ800-L1001-RG` resource group.

#### 2. Validation
- Confirmed that all Azure File Sync server registrations were removed.
- Confirmed that the cloud endpoint and sync group were deleted before removing the Storage Sync Service.
- Confirmed that the Azure storage account and resource group were removed successfully.

---

### Professional Insight
- **Cleanup Order Matters:** Azure File Sync resources should be removed in the correct sequence to avoid dependency errors.
- **Registered Servers:** Unregistering servers first ensures that related endpoints can be removed cleanly.
- **Cost Control:** Deleting the storage account and resource group ensures the lab no longer incurs Azure charges.
