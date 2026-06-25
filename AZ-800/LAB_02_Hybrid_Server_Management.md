# AZ-800: Administering Windows Server Hybrid Core Infrastructure

## Lab 2: Implementing integration between AD DS and Microsoft Entra ID

This lab focused on preparing Microsoft Entra ID for hybrid identity integration by creating a tenant, adding a custom domain, creating a Global Administrator account, and validating sign-in readiness.

---

## Lab Setup

### Task 1: Create a new Microsoft Entra tenant
In this task, I created a new Microsoft Entra tenant to prepare the environment for hybrid identity integration.

#### 1. Tenant Creation
- **Organization Name:** `Contoso Organization`
- **Initial Domain Name:** `Contoso35501731.onmicrosoft.com`
- **Tenant Type:** `Microsoft Entra ID`

#### 2. Validation
- Confirmed the new tenant was created successfully.
- Noted the tenant creation completion message in Azure notifications.
- Waited for directory synchronization to initialize before proceeding with later integration tasks.

---

## Exercise 1: Preparing Microsoft Entra ID for AD DS integration

### Task 1: Create a custom domain in Azure
In this task, I added a custom domain to Microsoft Entra ID to prepare the tenant for identity integration.

#### 1. Custom Domain Configuration
- **Custom Domain Name:** `contoso.com`

#### 2. Validation
- Reviewed the DNS record types required for domain verification.
- Closed the pane without verifying the domain, since verification was not required for the lab.

---

<img width="1151" height="725" alt="lab2 e1-1" src="https://github.com/user-attachments/assets/2fe82b8d-d275-40dc-b434-61fcf4c65ff8" />

---

### Task 2: Create a user with the Global Administrator role
In this task, I created a new user account and assigned the Global Administrator role in Microsoft Entra ID.

#### 1. User Creation
- **User Principal Name:** `admin@contoso35501731.onmicrosoft.com`
- **Display Name:** `admin`
- **Usage Location:** `United States`

#### 2. Role Assignment
- Assigned the **Global administrator** directory role to the user.

#### 3. Validation
- Confirmed the new user was created successfully.
- Recorded the generated password for later sign-in use.
- Verified the user had the required administrative role.

---

<img width="1117" height="663" alt="lab2 e1-2" src="https://github.com/user-attachments/assets/5b2f6c97-cb68-4f08-a2ee-917c4b9b7ab2" />

---

### Task 3: Change the password for the user with the Global Administrator role
In this task, I signed in as the newly created Global Administrator and changed the temporary password to a complex password.

#### 1. Sign-in Process
- Signed out of the Azure portal.
- Selected **Use another account**.
- Signed in with the fully qualified user principal name.

#### 2. Password Update
- Entered the temporary password provided during user creation.
- Set a new complex password.
- Recorded the new password for later use in the lab.

#### 3. Validation
- Confirmed the password change was successful.
- Verified the Global Administrator account could sign in normally.

---

<img width="1152" height="706" alt="lab2 e1-3" src="https://github.com/user-attachments/assets/97dd690d-79f9-4dc2-b573-45c84d0215ec" />

---

### Professional Insight
- **Tenant Preparation:** Creating a dedicated tenant provides a clean foundation for hybrid identity testing and administration.
- **Custom Domains:** Adding a custom domain supports a more realistic enterprise identity structure, even when full verification is not required for the lab.
- **Privilege Management:** Creating a separate Global Administrator account improves administrative separation and makes role assignment clearer during hybrid setup.

---

## Exercise 2: Preparing on-premises AD DS for Microsoft Entra ID integration

### Task 1: Install IdFix
In this task, I downloaded and prepared the IdFix tool for identifying and correcting directory synchronization issues.

#### 1. Tool Preparation
- **Tool Name:** `IdFix`
- **Purpose:** `Identify and remediate directory synchronization errors`
- **Target Host:** `SEA-ADM1`

#### 2. Validation
- Confirmed the IdFix tool was available for use.
- Verified the tool could be launched from the lab environment.

---

<img width="1036" height="688" alt="lab2 e2-1" src="https://github.com/user-attachments/assets/816f20ed-b1f7-454a-9150-4968adf089a4" />

---

### Task 2: Run IdFix and review directory issues
In this task, I ran IdFix against the on-premises Active Directory environment to identify objects that could cause synchronization problems with Microsoft Entra ID.

#### 1. Directory Review
- **Scope:** `On-premises AD DS`
- **Issues Reviewed:** `UPN, attribute, and naming conflicts`
- **Action:** `Assess and correct synchronization blockers`

#### 2. Validation
- Reviewed the IdFix results for directory issues.
- Identified any users or objects that required updates before Microsoft Entra Connect could be installed.

---

### Task 3: Correct UPNs for synchronization readiness
In this task, I ensured that user UPNs matched the Microsoft Entra tenant’s custom domain name so that synchronization would work correctly.

#### 1. UPN Alignment
- **Custom Domain:** `contoso.com`
- **User Principal Name Format:** `user@contoso.com`

#### 2. Validation
- Confirmed user UPNs were aligned with the tenant domain.
- Verified that the directory was ready for Microsoft Entra Connect configuration.

---

### Professional Insight
- **Directory Hygiene:** IdFix helps detect identity attribute issues before synchronization, reducing errors during Microsoft Entra Connect deployment.
- **UPN Consistency:** Matching on-premises UPNs to the verified or intended custom domain improves hybrid sign-in consistency.
- **Sync Readiness:** Preparing AD DS before installing Microsoft Entra Connect avoids common synchronization failures later in the lab.

---

## Exercise 3: Downloading, installing, and configuring Microsoft Entra Connect

### Task 1: Install and configure Microsoft Entra Connect
In this task, I downloaded Microsoft Entra Connect, installed it on SEA-ADM1, and configured express settings for hybrid identity synchronization.

#### 1. Installation Preparation
- **Target Server:** `SEA-ADM1`
- **Installation Tool:** `Microsoft Entra Connect`
- **Installation Mode:** `Express Settings`

#### 2. Configuration Steps
- Opened the Microsoft Entra Connect page in the Azure portal.
- Downloaded the installation binaries.
- Accepted the license terms and privacy notice.
- Selected **Use express settings**.
- Signed in with the Microsoft Entra ID Global Administrator account created in Exercise 1.
- Entered the on-premises AD DS credentials provided by the instructor.

#### 3. Microsoft Entra ID Sign-in Configuration
- Verified that the newly added domain appeared in the list of Active Directory UPN suffixes.
- Selected **Continue without matching all UPN suffixes to verified domains**.
- Reviewed the **Ready to configure** page.
- Started the installation.

#### 4. Validation
- Confirmed the installer completed the configuration process successfully.
- Verified that Microsoft Entra Connect was installed and configured on SEA-ADM1.
- Confirmed the tenant synchronization readiness requirement was respected before installation.

---

<img width="1162" height="767" alt="lab2 e3" src="https://github.com/user-attachments/assets/f8f93578-36d6-40b9-b6fe-15ccabb6718d" />

---

### Professional Insight
- **Hybrid Identity Sync:** Microsoft Entra Connect is the core tool used to synchronize on-premises AD DS identities with Microsoft Entra ID.
- **Credential Separation:** Using separate Microsoft Entra and AD DS credentials is a normal part of hybrid identity deployment.
- **Domain Planning:** Ensuring the UPN suffix appears in the sign-in configuration helps avoid later synchronization and sign-in issues.

---

## Exercise 4: Verifying integration between AD DS and Microsoft Entra ID

### Task 1: Verify synchronization in the Azure portal
In this task, I confirmed that directory synchronization was working by reviewing synced users and groups in the Microsoft Entra ID portal.

#### 1. Portal Verification
- **Portal:** `Azure portal`
- **Service:** `Microsoft Entra ID`
- **Sync Source:** `Active Directory`

#### 2. Validation
- Refreshed the Microsoft Entra Connect page.
- Reviewed the **Provision from Active Directory** section.
- Opened the **Users** page and observed synced users.
- Opened the **Groups** page and reviewed synced groups.

---

<img width="1132" height="803" alt="lab2 e4-1" src="https://github.com/user-attachments/assets/efd513e4-e883-49c9-bd55-2163e267efee" />

---

### Task 2: Verify synchronization in the Synchronization Service Manager
In this task, I reviewed synchronization activity and connector information in the Synchronization Service Manager.

#### 1. Synchronization Review
- **Tool:** `Synchronization Service Manager`
- **View:** `Operations`
- **Connectors:** `AD DS` and `Microsoft Entra tenant`

#### 2. Validation
- Reviewed recent sync operations in the Operations tab.
- Confirmed that two connectors were present.
- Verified that one connector represented the on-premises AD DS environment and the other represented Microsoft Entra ID.

---

<img width="1136" height="696" alt="lab2 e4-2" src="https://github.com/user-attachments/assets/10c8992e-e9f6-44d2-a0cf-93a7bfbb08d2" />

---

### Task 3: Update a user account in Active Directory
In this task, I updated an existing Active Directory user so that the change could be synchronized to Microsoft Entra ID.

#### 1. User Update
- **User:** `Sumesh Rajan`
- **OU:** `Sales`
- **Updated Attribute:** `Job Title`
- **New Value:** `Manager`

#### 2. Validation
- Opened the user properties in Active Directory Users and Computers.
- Updated the **Job Title** field under the Organization tab.
- Confirmed the change was saved successfully.

---

<img width="1112" height="791" alt="lab2 e4-3" src="https://github.com/user-attachments/assets/79437f6f-9a00-467e-a964-db147835c260" />

---

### Task 4: Create a user account in Active Directory
In this task, I created a new Active Directory user account in the Sales OU so that the account could later be synchronized to Microsoft Entra ID.

#### 1. New User Information
- **First Name:** `Jordan`
- **Last Name:** `Mitchell`
- **User Logon Name:** `Jordan`
- **Password:** `Pa55w.rd`
- **OU:** `Sales`

#### 2. Validation
- Confirmed the new user account was created successfully.
- Verified the account was placed in the correct OU.

---

### Task 5: Sync changes to Microsoft Entra ID
In this task, I forced a synchronization cycle so the changes made in Active Directory would be replicated to Microsoft Entra ID.

#### 1. Synchronization Command
```powershell
Start-ADSyncSyncCycle
```

#### 2. Validation
- Started a synchronization cycle from PowerShell as Administrator.
- Noted that synchronization may take time before the changes appear in Microsoft Entra ID.
- Confirmed the sync cycle was triggered successfully.

---

<img width="1096" height="718" alt="lab2 e4-4" src="https://github.com/user-attachments/assets/89e686b2-7ce4-4a34-9125-1fd326c55c0d" />

---

### Task 6: Verify changes in Microsoft Entra ID
In this task, I verified that the Active Directory updates were reflected in Microsoft Entra ID after synchronization completed.

#### 1. User Verification: Sumesh Rajan
- Searched for `Sumesh` in the Microsoft Entra ID Users page.
- Opened the user properties page.
- Verified that the **Job title** attribute was synced from Active Directory.

#### 2. User Verification: Jordan Mitchell
- Searched for `Jordan` in the Microsoft Entra ID Users page.
- Opened the user properties page.
- Reviewed the synced account attributes from Active Directory.

#### 3. Validation
- Confirmed the updated user attribute appeared in Microsoft Entra ID.
- Confirmed the newly created user account was also present after synchronization.

---
<img width="1152" height="812" alt="lab2 e4-5" src="https://github.com/user-attachments/assets/c2765386-566b-49d0-94a5-42d463c2f016" />


---

### Professional Insight
- **Synchronization Monitoring:** Reviewing both the portal and the Synchronization Service Manager helps verify that hybrid identity sync is functioning correctly.
- **Attribute Flow:** Updating a user attribute such as Job Title in AD DS demonstrates how on-premises directory data flows into Microsoft Entra ID.
- **Hybrid Identity Readiness:** Forcing a sync cycle with PowerShell is a useful administrative step when you need changes to appear sooner in the cloud.

--- 

## Exercise 5: Implementing Microsoft Entra ID integration features in AD DS

### Task 1: Enable self-service password reset in Azure
In this task, I activated the Microsoft Entra ID P2 trial and reviewed the password reset configuration options in the Azure portal.

#### 1. License Activation
- **License:** `Microsoft Entra ID P2`
- **Assigned User:** `Microsoft Entra ID Global Administrator`

#### 2. Password Reset Review
- Opened the **Password reset** page in Microsoft Entra ID.
- Reviewed the available scope settings for password reset.
- Left the password reset feature disabled as required by the lab.

#### 3. Validation
- Confirmed the P2 trial was activated successfully.
- Confirmed the Global Administrator user had the required license assigned.
- Reviewed the self-service password reset configuration without enabling it.

---

<img width="1145" height="656" alt="lab2 e5-1" src="https://github.com/user-attachments/assets/7e7f9624-939d-4c47-a190-af05b01cf03e" />

---

### Task 2: Enable password writeback in Microsoft Entra Connect
In this task, I enabled password writeback in Microsoft Entra Connect so cloud password changes can be written back to Active Directory.

#### 1. Configuration
- **Tool:** `Microsoft Entra Connect`
- **Optional Feature:** `Password writeback`

#### 2. Configuration Steps
- Opened **Configure** in Microsoft Entra Connect.
- Selected **Customize synchronization options**.
- Signed in with the Microsoft Entra ID Global Administrator account.
- Enabled **Password writeback** under Optional Features.
- Reviewed the **Ready to configure** page.
- Selected **Configure**.

#### 3. Validation
- Confirmed the configuration completed successfully.
- Closed Microsoft Entra Connect after the changes were applied.

---

<img width="1177" height="793" alt="lab2 e5-2" src="https://github.com/user-attachments/assets/90bd7f69-5db2-4f1f-b595-6c7988e211b2" />

---

### Task 3: Enable pass-through authentication in Microsoft Entra Connect
In this task, I changed the Microsoft Entra Connect user sign-in method to pass-through authentication and enabled seamless single sign-on.

#### 1. Sign-in Configuration
- **Sign-in Method:** `Pass-through authentication`
- **Seamless Single Sign-on:** `Enabled`

#### 2. Configuration Steps
- Opened **Change user sign-in** in Microsoft Entra Connect.
- Signed in with the Microsoft Entra ID Global Administrator account.
- Selected **Pass-through authentication**.
- Kept **Enable single sign-on** selected.
- Entered the forest credentials provided by the instructor.
- Reviewed the **Ready to configure** page.
- Selected **Configure**.

#### 3. Validation
- Confirmed the configuration completed successfully.
- Closed Microsoft Entra Connect after the changes were applied.

---

<img width="1157" height="781" alt="lab2 e5-3" src="https://github.com/user-attachments/assets/acde7a37-d390-498e-931e-feb8814b4186" />

---

### Task 4: Verify pass-through authentication in Azure
In this task, I reviewed the Microsoft Entra Connect status pages in the Azure portal to verify that pass-through authentication was enabled.

#### 1. Portal Review
- **Page:** `Microsoft Entra Connect`
- **Sign-in Method:** `Pass-through authentication`
- **Additional Setting:** `Seamless single sign-on`

#### 2. Validation
- Reviewed the **User Sign-In** section.
- Opened the **Seamless single sign-on** page and reviewed the on-premises domain name.
- Opened the **Passthrough Authentication** page.
- Reviewed the list of authentication agents registered in the tenant.

---

<img width="1151" height="780" alt="lab2 e5-4" src="https://github.com/user-attachments/assets/c4bc8457-f468-4bec-a555-73150955fa3e" />

---

### Task 5: Install and register the Microsoft Entra ID Password Protection proxy service and DC agent
In this task, I downloaded the Microsoft Entra Password Protection installers, installed the proxy service on SEA-SVR1, installed the DC agent on SEA-DC1, and registered both components with Active Directory.

#### 1. Installer Files
- **Proxy Installer:** `AzureADPasswordProtectionProxySetup.exe`
- **DC Agent Installer:** `AzureADPasswordProtectionDCAgentSetup.msi`

#### 2. Installation Summary
- Unblocked downloaded files in PowerShell.
- Copied the proxy installer to `SEA-SVR1`.
- Installed the proxy service on `SEA-SVR1`.
- Copied the DC agent installer to `SEA-DC1`.
- Installed the DC agent on `SEA-DC1`.
- Restarted the domain controller after installation.

#### 3. Service Validation
```powershell
Get-Service -Computer SEA-SVR1 -Name AzureADPasswordProtectionProxy | fl
Get-Service -Computer SEA-DC1 -Name AzureADPasswordProtectionDCAgent | fl
```

- Confirmed both services were running.

#### 4. Proxy Registration
```powershell
Enter-PSSession -ComputerName SEA-SVR1
Register-AzureADPasswordProtectionProxy -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
Exit-PSSession

Enter-PSSession -ComputerName SEA-DC1
Register-AzureADPasswordProtectionForest -AccountUpn <Azure_AD_Global_Admin> -AuthenticateUsingDeviceCode
Exit-PSSession
```

- Authenticated with the Microsoft Entra ID Global Administrator account.
- Registered the proxy service with Active Directory.
- Registered the forest for Microsoft Entra Password Protection.

---

<img width="1898" height="788" alt="lab2 e5-5" src="https://github.com/user-attachments/assets/90103d5d-f814-417a-b25a-ea637dc6af79" />

---

### Task 6: Enable password protection in Azure
In this task, I enabled password protection in Microsoft Entra ID and configured a custom banned password list for the organization.

#### 1. Password Protection Configuration
- **Custom Banned Password List:** `Contoso`, `London`
- **Windows Server Active Directory Protection:** `Enabled`
- **Mode:** `Audit`

#### 2. Validation
- Enabled **Enforce custom list**.
- Entered the organization-specific banned words.
- Verified password protection for Windows Server Active Directory was enabled.
- Confirmed the mode was set to **Audit**.
- Saved the configuration successfully.

---

<img width="1102" height="776" alt="lab2 e5-6" src="https://github.com/user-attachments/assets/e65b40a4-cc03-4a21-b56d-04c777affd2c" />


---

### Professional Insight
- **Password Writeback:** This feature allows cloud-based password resets to flow back to on-premises AD DS, which is essential for hybrid self-service password reset.
- **Pass-through Authentication:** PTA keeps authentication decisions on-premises while still integrating with Microsoft Entra ID.
- **Password Protection:** Microsoft Entra Password Protection helps reduce weak password usage in hybrid environments by enforcing banned password policies.

---

## Exercise 6: Cleaning up

### Task 1: Uninstall Microsoft Entra Connect
In this task, I removed Microsoft Entra Connect from SEA-ADM1 using Control Panel.

#### 1. Uninstall Steps
- **Target Server:** `SEA-ADM1`
- **Tool Removed:** `Microsoft Entra Connect`

#### 2. Validation
- Opened **Control Panel**.
- Used **Uninstall or change a program** to locate Microsoft Entra Connect.
- Uninstalled the application successfully.
- Confirmed the components were removed after refresh.

---

<img width="1075" height="777" alt="lab2 e6-1" src="https://github.com/user-attachments/assets/76aab92c-c5e3-48bc-80ce-c5cc8efcbeee" />

---

### Task 2: Disable directory synchronization in Azure
In this task, I used Microsoft Graph PowerShell to disable directory synchronization for the Microsoft Entra tenant.

#### 1. Microsoft Graph Installation
```powershell
Install-Module -Name Microsoft.Graph -Force
```

#### 2. Connect to Microsoft Graph
```powershell
Connect-MgGraph -Scopes "Organization.ReadWrite.All"
```

#### 3. Disable Directory Synchronization
```powershell
$OrgID = (Get-MgOrganization).Id
$params = @{ onPremisesSyncEnabled = $false }
Update-MgOrganization -OrganizationId $OrgID -BodyParameter $params
```

#### 4. Verify the Result
```powershell
Get-MgOrganization | Select-Object DisplayName, OnPremisesSyncEnabled
```

#### 5. Validation
- Confirmed that **OnPremisesSyncEnabled** was set to `False`.
- Noted that synchronized users may take up to 72 hours to become fully cloud-only accounts.
- Verified that directory synchronization had been disabled successfully.

---

<img width="1132" height="725" alt="lab2 e6-2" src="https://github.com/user-attachments/assets/c6116bc2-42c0-418e-a517-3a7f28c72898" />

---

### Professional Insight
- **Cleanup Matters:** Uninstalling Microsoft Entra Connect removes the local synchronization component before disabling tenant sync.
- **Graph-Based Administration:** Microsoft Graph PowerShell is the current method for managing directory synchronization settings.
- **Cloud-Only Transition:** Disabling synchronization does not instantly convert all synced objects; the transition can take time.
