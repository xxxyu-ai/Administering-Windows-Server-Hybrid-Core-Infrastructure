# AZ-800: Administering Windows Server Hybrid Core Infrastructure

## Lab 1: Implementing Identity Services and Group Policy

This lab focused on deploying a new domain controller on Windows Server Core and managing objects in Active Directory Domain Services.

---

## Exercise 1: Deploying a new domain controller on Server Core

### Task 1: Deploy AD DS on a new Windows Server Core server
In this task, I installed the Active Directory Domain Services role on SEA-SVR1 and verified that the required role components were installed successfully.

#### 1. Install AD DS Role
- **Target Server:** `SEA-SVR1`
- **Role:** `Active Directory Domain Services`
- **Management Host:** `SEA-ADM1`

```powershell
Install-WindowsFeature -Name AD-Domain-Services -ComputerName SEA-SVR1
Get-WindowsFeature -ComputerName SEA-SVR1
```

#### 2. Verification
- Confirmed that **Active Directory Domain Services** was selected.
- Confirmed that **AD DS and AD LDS Tools** under RSAT were installed.
- Verified the Active Directory PowerShell module was available.

---

### Task 2: Prepare the AD DS installation and promote a remote server
In this task, I prepared SEA-SVR1 for domain controller promotion and completed the promotion by using the generated PowerShell command.

#### 1. Promotion Preparation
- Added `SEA-SVR1` to Server Manager.
- Opened the post-deployment configuration from the notification flag.
- Selected **Add a domain controller to an existing domain**.
- Used the `Contoso.com` domain credentials provided by the instructor.
- Confirmed the DNS server and Global Catalog options were enabled.

#### 2. PowerShell-Based Promotion
```powershell
Invoke-Command -ComputerName SEA-SVR1 {Install-ADDSDomainController -NoGlobalCatalog:$false -CreateDnsDelegation:$false -Credential (Get-Credential) -CriticalReplicationOnly:$false -DatabasePath "C:\Windows\NTDS" -DomainName "Contoso.com" -InstallDns:$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:$false -SiteName "Default-First-Site-Name" -SysvolPath "C:\Windows\SYSVOL" -Force:$true}
```

#### 3. Validation
- Waited for the command to complete successfully.
- Confirmed that `SEA-SVR1` restarted after promotion.
- Verified that `SEA-SVR1` appeared under the AD DS node in Server Manager.
- Confirmed the warning notification disappeared.

---

### Task 3: Manage objects in AD DS
In this task, I created an OU, user account, security group, and local administrator membership in Active Directory.

#### 1. Organizational Unit Creation
```powershell
New-ADOrganizationalUnit -Name "Seattle" -Path "DC=contoso,DC=com" -ProtectedFromAccidentalDeletion $true -Server SEA-DC1.contoso.com
```

#### 2. User Account Creation
```powershell
New-ADUser -Name Ty -DisplayName 'Ty Carlson' -GivenName Ty -Surname Carlson -Path 'OU=Seattle,DC=contoso,DC=com'
```

#### 3. Password Configuration and Enablement
```powershell
Set-ADAccountPassword Ty
Enable-ADAccount Ty
```

#### 4. Group Creation and Membership
```powershell
New-ADGroup SeattleBranchUsers -Path 'OU=Seattle,DC=contoso,DC=com' -GroupScope Global -GroupCategory Security
Add-ADGroupMember -Identity SeattleBranchUsers -Members Ty
Get-ADGroupMember -Identity SeattleBranchUsers
```

#### 5. Local Administrators Group Membership
```powershell
Add-LocalGroupMember -Group 'Administrators' -Member 'CONTOSO\Ty'
```

#### 6. Validation
- Confirmed that the **Seattle** OU was created successfully.
- Confirmed that the **Ty Carlson** user account was created and enabled.
- Confirmed that **SeattleBranchUsers** contained the Ty account.
- Confirmed that `CONTOSO\Ty` was added to the local Administrators group.

---

### Evidence
> **[Picture]**
> *Capture Server Manager showing SEA-SVR1 added as a domain controller and the Active Directory Users and Computers console showing the Seattle OU, Ty user, and SeattleBranchUsers group.*

---

### Professional Insight
- **Domain Controller Deployment:** Promoting a remote Server Core system through PowerShell is an efficient and repeatable way to deploy AD DS.
- **Directory Management:** Creating OUs, users, and groups with PowerShell supports consistent administration and automation.
- **Access Control:** Adding a user to the local Administrators group can be necessary for delegated administrative access in lab environments.

---

## Exercise 2: Configuring Group Policy

### Task 1: Create and edit a GPO
In this task, I created a new domain GPO named CONTOSO Standards and configured user-based policy settings for registry access and screen saver behavior.

#### 1. Create the GPO
- **GPO Name:** `CONTOSO Standards`
- **Scope:** `Contoso.com domain`
- **Management Tool:** `Group Policy Management`

#### 2. Configure User Settings
```powershell
# This task was completed through the Group Policy Management Editor GUI.
# The following policies were configured:
# - Prevent access to registry editing tools: Enabled
# - Screen saver timeout: Enabled, 600 seconds
# - Password protect the screen saver: Enabled
```

#### 3. Validation
- Confirmed the GPO was created under **Group Policy Objects**.
- Confirmed the required policy settings were enabled in the editor.

---

### Task 2: Link the GPO
In this task, I linked the CONTOSO Standards GPO to the Contoso.com domain so the policy would apply at the domain level.

#### 1. GPO Link
- **Linked GPO:** `CONTOSO Standards`
- **Linked To:** `Contoso.com domain`

#### 2. Validation
- Confirmed the GPO appeared as linked under the domain.
- Verified the link was active and available for inheritance.

---

### Task 3: Review the effects of the GPO's settings
In this task, I validated that the CONTOSO Standards GPO applied the expected user restrictions and personalization settings.

#### 1. Policy Effect Verification
- **Registry Editing:** Disabled by policy
- **Screen Saver Timeout:** Forced to `600 seconds`
- **Password Protection:** Enabled

#### 2. Validation Steps
- Opened **Control Panel** and confirmed the screen saver settings were enforced.
- Signed in as `CONTOSO\Ty` and verified that the timeout value could not be changed.
- Ran `regedit` and confirmed the message: `Registry editing has been disabled by your administrator.`

---

### Task 4: Create and link the required GPOs
In this task, I created a new OU-linked GPO named Seattle Application Override and configured it to override the domain-level screen saver setting.

#### 1. OU-linked GPO
- **GPO Name:** `Seattle Application Override`
- **Linked To:** `Seattle OU`

#### 2. Policy Configuration
- **Screen saver timeout:** `Disabled`

#### 3. Validation
- Confirmed the GPO was created and linked to the Seattle OU.
- Verified the policy editor showed the screen saver timeout setting as disabled.

---

### Task 5: Verify the order of precedence
In this task, I reviewed Group Policy inheritance to confirm which GPO had higher precedence.

#### 1. Inheritance Review
- **Domain GPO:** `CONTOSO Standards`
- **OU GPO:** `Seattle Application Override`

#### 2. Validation
- Confirmed the Seattle Application Override GPO had higher precedence.
- Verified that the OU-level GPO would override the domain-level screen saver timeout setting.

---

### Task 6: Configure the scope of a GPO with security filtering
In this task, I configured security filtering so the Seattle Application Override GPO applied only to the intended users and computer.

#### 1. Security Filtering Configuration
- Removed **Authenticated Users**
- Added **SeattleBranchUsers**
- Added **SEA-ADM1** computer account

#### 2. Validation
- Confirmed the GPO security filtering list contained only the intended security principals.
- Verified that computer read access requirements were considered when adding the computer account.

---

### Task 7: Verify the application of settings
In this task, I used Group Policy Modeling to simulate policy application and verify the resulting effective settings.

#### 1. Modeling Configuration
- **User:** `CONTOSO\Ty`
- **Computer:** `CONTOSO\SEA-ADM1`
- **User Security Group:** `CONTOSO\SeattleBranchUsers`

#### 2. Validation
- Confirmed the modeling report showed **Seattle Application Override** as the winning GPO.
- Verified that the **Screen saver timeout** setting was disabled in the final policy result.
- Confirmed the report reflected the expected inheritance and filtering behavior.

---

### Evidence
> **[Picture]**
> *Capture the Group Policy Management console showing the GPO links, security filtering, and the Group Policy Modeling report with Seattle Application Override as the winning GPO.*

---

### Professional Insight
- **Policy Design:** Domain-linked GPOs provide a baseline policy, while OU-linked GPOs can override specific settings for targeted users.
- **Scope Control:** Security filtering is useful when you want a GPO to apply only to a specific group or computer.
- **Validation:** Group Policy Modeling helps confirm the effective policy before deployment or troubleshooting in production.
