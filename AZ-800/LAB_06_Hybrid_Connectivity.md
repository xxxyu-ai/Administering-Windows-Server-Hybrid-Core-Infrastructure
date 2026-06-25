# AZ-800: Administering Windows Server Hybrid Core Infrastructure

## Lab6: Deploying and configuring Windows Server on Azure VMs

This lab focused on authoring an ARM template for Azure VM deployment, enabling Microsoft Defender for Cloud enhanced security, and downloading the generated template and parameter files for later customization.

---

## Exercise 1: Authoring ARM templates for Azure VM deployment

### Task 1: Connect to your Azure subscription and enable enhanced security of Microsoft Defender for Cloud
In this task, I connected to the Azure subscription and enabled Microsoft Defender for Cloud enhanced security features.

#### 1. Azure Security Configuration
- **Service:** `Microsoft Defender for Cloud`
- **Enhanced Security:** `Enabled`
- **Agent Installation:** `Automatic`

#### 2. Validation
- Signed in to the Azure portal with an Owner account.
- Opened the Microsoft Defender for Cloud page.
- Enabled enhanced security.
- Enabled automatic Microsoft Defender for Cloud agent installation.

---

<img width="1163" height="767" alt="lab6 e1-1" src="https://github.com/user-attachments/assets/8c382e0a-ce98-4261-b371-9399f8f4c184" />

---

### Task 2: Generate an ARM template and parameters files by using the Azure portal
In this task, I created the initial Azure VM configuration in the portal so that Azure could generate an ARM template and parameter file.

#### 1. VM Configuration
- **Subscription:** `Azure subscription used in this lab`
- **Resource Group:** `AZ800-L0601-RG`
- **Virtual Machine Name:** `az800L06-vm0`
- **Region:** `<Azure region>`
- **Availability Options:** `No infrastructure redundancy required`
- **Image:** `Windows Server 2022 Datacenter: Azure Edition - Gen2`
- **Azure Spot Instance:** `No`
- **Size:** `Standard_D2s_v3`
- **Username:** `Student`
- **Password:** `Pa55w.rd1234`
- **Public Inbound Ports:** `None`
- **Existing Windows Server License:** `No`
- **OS Disk Type:** `Standard HDD`
- **Virtual Network:** `az800L06-vnet`
- **Address Range:** `10.60.0.0/20`
- **Subnet Name:** `subnet0`
- **Subnet Range:** `10.60.0.0/24`
- **Public IP:** `None`
- **NSG:** `None`
- **Accelerated Networking:** `Off`
- **Load Balancing Solution:** `No`
- **Boot Diagnostics:** `Enabled with managed storage account`

#### 2. Validation
- Stepped through the Create a virtual machine wizard.
- Reached the **Review + Create** page.
- Did not deploy the VM manually.
- Confirmed the portal generated an ARM template and parameters file.

---

<img width="1133" height="792" alt="lab6 e1-2" src="https://github.com/user-attachments/assets/39fb5c5c-c0bc-400a-8e47-e340c74fd2f3" />

---

### Task 3: Download the ARM template and parameters files from the Azure portal
In this task, I downloaded the generated template package and saved it in the lab file folder for later use.

#### 1. File Download
- **Download Location:** `C:\Labfiles\Mod06`
- **Files:** ARM template and parameter files

#### 2. Validation
- Downloaded the template for automation from the portal.
- Copied the downloaded files to `C:\Labfiles\Mod06`.
- Closed the Create a virtual machine page after downloading.

---

<img width="1117" height="678" alt="lab6 e1-3" src="https://github.com/user-attachments/assets/f5e04399-a881-4af4-90c9-42e52e82df0d" />

---

### Professional Insight
- **Infrastructure as Code:** ARM templates allow repeatable and auditable VM deployment.
- **Security Baseline:** Enabling Defender for Cloud early helps ensure secure-by-default VM deployment.
- **Template Reuse:** Exporting the template from the portal provides a practical starting point for automation and later customization.

---

## Exercise 2: Modifying ARM templates to include VM extension-based configuration

### Task 1: Review the ARM template and parameters files for Azure VM deployment
In this task, I reviewed the exported ARM template and parameters file that were generated from the Azure portal.

#### 1. File Review
- **Template File:** `template.json`
- **Parameters File:** `parameters.json`

#### 2. Validation
- Extracted the contents of the downloaded archive into `C:\Labfiles\Mod06`.
- Opened `template.json` in Notepad and reviewed the structure.
- Opened `parameters.json` in Notepad and reviewed the parameter values.
- Kept the template file open for modification.

---

<img width="1188" height="817" alt="lab6 e2-1" src="https://github.com/user-attachments/assets/cdf28b85-1fcd-4675-8276-f400edeb26b3" />

---

### Task 2: Add an Azure VM extension section to the existing template
In this task, I added a Custom Script Extension resource to the ARM template so the VM could be configured automatically during deployment.

#### 1. Extension Configuration
- **Resource Type:** `Microsoft.Compute/virtualMachines/extensions`
- **Extension Name:** `customScriptExtension`
- **Publisher:** `Microsoft.Compute`
- **Extension Type:** `CustomScriptExtension`
- **Type Handler Version:** `1.7`

#### 2. Custom Script Command
```json
"commandToExecute": "powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools && powershell.exe remove-item 'C:\\inetpub\\wwwroot\\iisstart.htm' && powershell.exe Add-Content -Path 'C:\\inetpub\\wwwroot\\iisstart.htm' -Value $('Hello World from ' + $env:computername)"
```

#### 3. Template Modification
```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "[concat(parameters('virtualMachineName'), '/customScriptExtension')]",
  "apiVersion": "2018-06-01",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', parameters('virtualMachineName'))]"
  ],
  "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.7",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "commandToExecute": "powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools && powershell.exe remove-item 'C:\\inetpub\\wwwroot\\iisstart.htm' && powershell.exe Add-Content -Path 'C:\\inetpub\\wwwroot\\iisstart.htm' -Value $('Hello World from ' + $env:computername)"
    }
  }
}
```

#### 4. Validation
- Inserted the extension section directly under the `"resources": [` line.
- Saved the updated `template.json`.
- Closed the file after making the change.

---

<img width="947" height="517" alt="lab6 e2-2" src="https://github.com/user-attachments/assets/eff3de1a-23e0-4e49-a690-b57a34dc5d01" />

---

### Professional Insight
- **VM Extensions:** Azure VM extensions let you perform post-deployment configuration automatically.
- **Custom Script Extension:** This extension is useful for installing roles and modifying files without manual login.
- **Infrastructure as Code:** Adding configuration into the template improves repeatability and reduces deployment drift.

---

## Exercise 3: Deploying Azure VMs running Windows Server by using ARM templates

### Task 1: Deploy an Azure VM by using an ARM template
In this task, I deployed the Azure VM from the modified ARM template in the Azure portal.

#### 1. Deployment Settings
- **Subscription:** `Azure subscription used in this lab`
- **Resource Group:** `AZ800-L0601-RG`
- **Region:** `<Azure region>`
- **Admin Password:** `Pa55w.rd1234`

#### 2. Validation
- Opened **Custom deployment** in the Azure portal.
- Selected **Build your own template in the editor**.
- Loaded the template and parameter files.
- Deployed the template successfully.
- Waited for the deployment to complete.

---

<img width="1118" height="660" alt="lab6 e3-1" src="https://github.com/user-attachments/assets/53582e91-c25c-4174-bcb6-b41f832cf93d" />

---

### Task 2: Review results of the Azure VM deployment
In this task, I reviewed the deployed resources and verified that the Custom Script Extension completed successfully.

#### 1. Resource Group Review
- **Resource Group:** `AZ800-L0601-RG`
- **Deployed VM:** `az800L06-vm0`

#### 2. Extension Verification
- Opened the `az800L06-vm0` VM page.
- Verified that `customScriptExtension` was provisioned successfully.

#### 3. Deployment Verification
- Opened the **Deployments** section of the resource group.
- Reviewed the `Microsoft.Template` deployment record.
- Confirmed that the deployment matched the template used in the custom deployment.

---

<img width="1121" height="695" alt="lab6 e3-2" src="https://github.com/user-attachments/assets/0b4b2d90-2edc-4d66-adec-1993ad79bbf0" />

---

### Professional Insight
- **Template Deployment:** ARM templates make it possible to deploy both infrastructure and configuration in a repeatable way.
- **Extension Validation:** Reviewing extension status is important to confirm that post-deployment configuration completed successfully.
- **Deployment Auditability:** Azure deployment history provides a useful record for checking what was deployed and from which template.

---

## Exercise 4: Configuring administrative access to Azure VMs running Windows Server

This exercise focused on verifying Microsoft Defender for Cloud enhanced security settings and reviewing Just-in-time VM access settings for Azure VMs running Windows Server.

---

### Task 1: Verify the status of Azure Microsoft Defender for Cloud
In this task, I confirmed that Microsoft Defender for Cloud enhanced security features were enabled for the subscription.

#### 1. Security Review
- **Service:** `Microsoft Defender for Cloud`
- **Enhanced Security Features:** `Enabled`

#### 2. Validation
- Opened the Microsoft Defender for Cloud page in the Azure portal.
- Verified that enhanced security was enabled.

---

<img width="1143" height="798" alt="lab6 e4-1" src="https://github.com/user-attachments/assets/89a3e3fa-9fc3-4d3b-bfc5-87028b1f70c2" />

---

### Task 2: Review the Just-in-time VM access settings
In this task, I reviewed the JIT VM access configuration pages in Microsoft Defender for Cloud.

#### 1. JIT Review
- **Page:** `Microsoft Defender for Cloud | Workload protections`
- **Feature:** `Just-in-time VM access`

#### 2. Tabs Reviewed
- `Configured`
- `Not Configured`
- `Unsupported`

#### 3. Validation
- Reviewed the Just-in-time VM access settings.
- Noted that newly deployed VMs may take up to 24 hours to appear in the Unsupported tab.
- Continued to the next exercise without waiting.

---

<img width="1226" height="716" alt="lab6 e4-2" src="https://github.com/user-attachments/assets/5d5e36ba-bf68-4dfb-85ad-6f10a28b8a21" />

---

### Professional Insight
- **Defender for Cloud:** Enhanced security settings help enforce a stronger security baseline for Azure VMs.
- **Just-in-time Access:** JIT reduces the attack surface by limiting when management ports can be opened.
- **Operational Awareness:** Reviewing JIT status early helps identify whether VMs are ready for secure administrative access.

---

## Exercise 5: Configuring Windows Server security in Azure VMs

This exercise focused on creating and attaching a network security group, exposing HTTP access to the Azure VM, re-evaluating the VM for JIT access, and connecting to the VM through just-in-time VM access.

---

### Task 1: Create and configure an NSG
In this task, I created a network security group and added an inbound rule to allow HTTP traffic.

#### 1. NSG Configuration
- **Subscription:** `Azure subscription used in this lab`
- **Resource Group:** `AZ800-L0601-RG`
- **NSG Name:** `az800L06-vm0-nsg1`
- **Region:** `<Azure region used for az800L06-vm0>`

#### 2. Inbound Rule
- **Rule Name:** `AllowHTTPInBound`
- **Source:** `Any`
- **Source Port Ranges:** `*`
- **Destination:** `Any`
- **Service:** `HTTP`
- **Action:** `Allow`
- **Priority:** `300`

#### 3. Validation
- Confirmed the NSG was created successfully.
- Added the inbound HTTP allow rule.

---

<img width="1161" height="717" alt="lab6 e5-1" src="https://github.com/user-attachments/assets/b47b1c26-9c12-4f56-879a-6338ca96cd6e" />

---

### Task 2: Configure inbound HTTP access to an Azure VM
In this task, I associated the NSG with the VM network interface and assigned a public IP address so the web page could be reached over HTTP.

#### 1. Network Interface Configuration
- Associated the VM NIC with `az800L06-vm0-nsg1`.

#### 2. Public IP Configuration
- **Public IP Name:** `az800L06-vm0-pip1`
- **SKU:** `Standard`

#### 3. Validation
- Opened the public IP address from the lab VM browser.
- Verified the page displayed `Hello World from az800L06-vm0`.
- Attempted RDP to the same IP and confirmed the connection failed, as expected.

---

<img width="1167" height="412" alt="lab6 e5-2" src="https://github.com/user-attachments/assets/80a145fc-5c15-4aaf-b750-9b5a633446ba" />

---

### Task 3: Trigger re-evaluation of the JIT status of an Azure VM
In this task, I forced Microsoft Defender for Cloud to re-evaluate the VM so it would appear in the JIT configuration list.

#### 1. JIT Re-evaluation
- **Target VM:** `az800L06-vm0`

#### 2. Validation
- Opened the VM page in the Azure portal.
- Selected **Configuration**.
- Enabled **just-in-time VM access**.
- Opened Microsoft Defender for Cloud.
- Verified that `az800L06-vm0` appeared on the **Configured** tab.

---

<img width="1167" height="696" alt="lab6 e5-3" src="https://github.com/user-attachments/assets/e9de6cc5-3599-49bf-8590-952df555550c" />

---

### Task 4: Connect to the Azure VM via JIT VM access
In this task, I requested JIT access and connected to the Azure VM through Remote Desktop.

#### 1. Access Request
- Requested JIT VM access for `az800L06-vm0`.
- Waited for approval.

#### 2. Remote Desktop Connection
- Connected to the VM through RDP.
- **Username:** `Student`
- **Password:** `Pa55w.rd1234`

#### 3. Validation
- Confirmed that Remote Desktop access worked after JIT approval.
- Verified successful access to the operating system running on the Azure VM.
- Closed the Remote Desktop session.

---

<img width="1103" height="782" alt="lab6 e5-4" src="https://github.com/user-attachments/assets/d5d7debf-e7a2-4cbb-8422-6cbe1e3e1a6c" />

---

### Professional Insight
- **Network Security:** An NSG is the primary control for restricting inbound access to Azure VMs.
- **Just-in-Time Access:** JIT improves security by opening management ports only when needed and only for approved access windows.
- **Least Privilege:** Exposing only HTTP while denying RDP from the public internet reduces the VM attack surface.

---

## Exercise 6: Deprovisioning the Azure environment

This exercise focused on cleaning up the Azure resources created during the lab to avoid unnecessary charges.

---

### Task 1: Start a PowerShell session in Cloud Shell
In this task, I opened a PowerShell session in Azure Cloud Shell.

#### 1. Cloud Shell Access
- **Portal:** `Azure portal`
- **Shell Type:** `PowerShell`

#### 2. Validation
- Opened Cloud Shell successfully.
- Accepted the default settings if prompted for the first time.

---

<img width="1147" height="718" alt="lab6 e6" src="https://github.com/user-attachments/assets/85267ff1-6be7-4200-88f0-68cf58f15dc6" />

---

### Task 2: Identify all Azure resources provisioned in the lab
In this task, I listed and deleted the resource groups created throughout the lab.

#### 1. List Resource Groups
```powershell
Get-AzResourceGroup -Name 'AZ800-L06*'
```

#### 2. Delete Resource Groups
```powershell
Get-AzResourceGroup -Name 'AZ800-L06*' | Remove-AzResourceGroup -Force -AsJob
```

#### 3. Validation
- Confirmed that all matching resource groups were listed.
- Started asynchronous deletion of all matching resource groups.
- Noted that deletion continues in the background because of the `-AsJob` parameter.

---

### Professional Insight
- **Cost Control:** Deprovisioning unused lab resources helps avoid unnecessary Azure charges.
- **Efficient Cleanup:** PowerShell makes it easy to remove multiple lab resource groups at once.
- **Asynchronous Deletion:** The `-AsJob` switch allows deletion to continue without blocking the current session.
