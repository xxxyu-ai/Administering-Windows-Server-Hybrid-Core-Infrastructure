# AZ-800: Administering Windows Server Hybrid Core Infrastructure

## Lab: Using Windows Admin Center in hybrid scenarios

This lab focused on using Windows Admin Center in a hybrid environment, deploying Azure resources with ARM templates, and verifying that Windows Admin Center works consistently across on-premises and Azure-managed systems.

---

## Exercise 1: Provisioning Azure VMs running Windows Server

### Task 1: Create an Azure resource group by using an Azure Resource Manager template
In this task, I used Azure Cloud Shell to deploy a resource group for the lab environment.

#### 1. Resource Group Deployment
- **Resource Group Name:** `AZ800-L0401-RG`
- **Azure Region:** `East US`

#### 2. Validation
- Uploaded `L04-sub_template.json` to Cloud Shell.
- Created the resource group using `New-AzSubscriptionDeployment`.
- Confirmed the deployment completed successfully.

---

### Task 2: Create an Azure VM by using an Azure Resource Manager template
In this task, I deployed an Azure VM running Windows Server for use in the hybrid lab.

#### 1. VM Deployment
- Uploaded `L04-rg_template.json`.
- Uploaded `L04-rg_template.parameters.json`.
- Deployed the Azure VM using `New-AzResourceGroupDeployment`.

#### 2. Validation
- Confirmed the deployment completed successfully.
- Waited for the VM deployment to finish before continuing.

---

### Task 3: Add GatewaySubnet to the virtual network
In this task, I updated the Azure virtual network to support the Windows Admin Center gateway and hybrid connectivity.

#### 1. Virtual Network Configuration
- **Virtual Network:** `az800l04-vnet`
- **Subnet Name:** `GatewaySubnet`
- **Address Range:** `10.4.3.224/27`

#### 2. Validation
- Confirmed the GatewaySubnet was added successfully.
- Verified the virtual network was ready for hybrid connectivity scenarios.

---

### Evidence
> **[Picture]**
> *Capture the Cloud Shell deployment commands, the created resource group, the deployed Azure VM, and the virtual network showing the added GatewaySubnet.*

---

### Professional Insight
- **ARM Deployment:** Using templates makes the lab environment repeatable and consistent.
- **Hybrid Readiness:** A dedicated GatewaySubnet is required for Azure networking features that support hybrid connectivity.
- **Azure VM Provisioning:** Deploying the VM first provides the target environment for later Windows Admin Center testing.

---

## Exercise 2: Implementing hybrid connectivity by using the Azure Network Adapter

### Task 1: Register Windows Admin Center with Azure
In this task, I connected Windows Admin Center to the Azure subscription so that it could create the Azure Network Adapter resources.

#### 1. Windows Admin Center Preparation
- **Windows Admin Center URL:** `https://SEA-ADM1.contoso.com`
- **Target Server:** `SEA-ADM1`

#### 2. Registration Steps
- Started Windows PowerShell as Administrator.
- Downloaded and installed Windows Admin Center if needed.
- Opened Windows Admin Center in Microsoft Edge.
- Signed in with the instructor-provided credentials.
- Attempted to add an Azure Network Adapter.
- Registered Windows Admin Center to the Azure subscription when prompted.

#### 3. Validation
- Confirmed Windows Admin Center was linked to the Azure subscription.
- Verified the Azure registration step completed successfully.

---

### Task 2: Create an Azure Network Adapter
In this task, I created an Azure Network Adapter to establish hybrid connectivity between SEA-ADM1 and the Azure virtual network.

#### 1. Azure Network Adapter Settings
- **Subscription:** `Azure subscription used in this lab`
- **Location:** `eastus`
- **Virtual Network:** `az800l04-vnet`
- **Gateway Subnet:** `10.4.3.224/27`
- **Gateway SKU:** `VpnGw1`
- **Client Address Space:** `192.168.0.0/24`
- **Authentication Certificate:** `Auto-generated Self-signed root and client Certificate`

#### 2. Validation
- Created the Azure Network Adapter successfully.
- Confirmed that Windows Admin Center initiated the creation of the Azure virtual network gateway.
- Verified that the gateway began provisioning in the Azure portal.

#### 3. Azure Portal Review
- Opened the Azure portal.
- Confirmed that a new virtual network gateway with a name starting with `WAC-Created-vpngw-` was provisioning.
- Noted that gateway provisioning can take a long time, so I continued to the next exercise without waiting for completion.

---

### Evidence
> **[Picture]**
> *Capture Windows Admin Center showing Azure registration and Azure Network Adapter settings, and the Azure portal showing the WAC-created virtual network gateway being provisioned.*

---

### Professional Insight
- **Hybrid Connectivity:** Azure Network Adapter provides a quick way to create point-to-site connectivity between an on-premises server and Azure.
- **Windows Admin Center Integration:** Registering WAC with Azure is required before it can create Azure networking resources.
- **Provisioning Time:** Azure virtual network gateway creation can take a long time, so it is normal to continue while provisioning completes in the background.

---

## Exercise 3: Deploying Windows Admin Center gateway in Azure

### Task 1: Install Windows Admin Center gateway in Azure
In this task, I used Azure Cloud Shell to run the provisioning script that deploys a Windows Admin Center gateway VM in Azure.

#### 1. Script Preparation
- **Script File:** `Deploy-WACAzVM.ps1`
- **Resource Group:** `AZ800-L0401-RG`
- **Virtual Network:** `az800l04-vnet`
- **Subnet:** `subnet1`
- **NSG:** `az800l04-web-nsg`
- **Public IP Name:** `wac-public-ip`
- **VM Name:** `az800l04-vmwac`
- **Size:** `Standard_D2s_v3`

#### 2. Cloud Shell Commands
```powershell
Enable-AzureRmAlias -Scope Process

$rgName = 'AZ800-L0401-RG'
$vnetName = 'az800l04-vnet'
$nsgName = 'az800l04-web-nsg'
$subnetName = 'subnet1'
$location = '<Azure region>'
$pipName = 'wac-public-ip'
$size = 'Standard_D2s_v3'

$scriptParams = @{
  ResourceGroupName = $rgName
  Name = 'az800l04-vmwac'
  VirtualNetworkName = $vnetName
  SubnetName = $subnetName
  GenerateSslCert = $true
  size = $size
  PublicIPAddressName = $pipname
}

install-module pswsman
Disable-WSManCertVerification -All

./Deploy-WACAzVM.ps1 @scriptParams
```

#### 3. Validation
- Uploaded the provisioning script to Cloud Shell.
- Started the provisioning script successfully.
- Entered the local Administrator account name `Student`.
- Entered the password `Pa55w.rd1234`.
- Waited for the script to complete and recorded the generated fully qualified DNS name for the Azure VM hosting Windows Admin Center.

---

### Task 2: Review results of the script provisioning
In this task, I reviewed the Azure resource group and networking rules created by the provisioning script.

#### 1. Resource Group Review
- **Resource Group:** `AZ800-L0401-RG`
- **Provisioned VM:** `az800l04-vmwac`

#### 2. Network Security Review
- Opened the VM networking settings.
- Reviewed inbound rules allowing connectivity on:
  - **TCP 5986**
  - **TCP 443**

#### 3. Validation
- Confirmed that the resource group contained the expected resources.
- Verified that the VM had inbound rules for secure management and web access.

---

### Evidence
> **[Picture]**
> *Capture Cloud Shell showing the provisioning script execution, the generated WAC VM FQDN, and the Azure portal showing the resource group and inbound networking rules.*

---

### Professional Insight
- **Scripted Deployment:** Using a script makes the Windows Admin Center gateway deployment repeatable and consistent.
- **Secure Management:** Port 443 supports web access, and port 5986 supports PowerShell remoting over HTTPS.
- **Cloud Provisioning:** Reviewing the resource group and NSG confirms that the deployment created all required Azure components.

---

## Exercise 4: Verifying functionality of the Windows Admin Center gateway in Azure

### Task 1: Connect to the Windows Admin Center gateway running in Azure VM
In this task, I connected to the Windows Admin Center gateway running in the Azure VM and reviewed the server overview.

#### 1. Connection Details
- **Gateway VM:** `az800l04-vmwac`
- **WAC URL:** `https://<gateway-vm-fqdn>`
- **Credentials:** `Student / Pa55w.rd1234`

#### 2. Validation
- Opened the Windows Admin Center gateway in Microsoft Edge.
- Signed in successfully with the provided credentials.
- Selected `az800l04-vmwac [Gateway]` from All connections.
- Reviewed the Overview pane.

---

### Task 2: Enable PowerShell Remoting on an Azure VM
In this task, I enabled Windows Remote Management and PowerShell Remoting on the Azure VM so that it could be managed through Windows Admin Center.

#### 1. Target VM
- **Azure VM:** `az800l04-vm0`

#### 2. Run Command Actions
```powershell
winrm quickconfig -quiet
Set-NetFirewallRule -Name WINRM-HTTP-In-TCP-PUBLIC -RemoteAddress Any
Enable-PSRemoting -Force -SkipNetworkProfileCheck
```

#### 3. Validation
- Confirmed Windows Remote Management was enabled.
- Confirmed the firewall rule was updated for WinRM access.
- Confirmed PowerShell Remoting was enabled successfully.

---

### Task 3: Connect to an Azure VM by using the Windows Admin Center gateway running in Azure VM
In this task, I connected to the Azure VM through the Windows Admin Center gateway that was deployed in Azure.

#### 1. Connection Details
- **Target Azure VM:** `az800l04-vm0`
- **Connection Method:** `Windows Admin Center gateway`

#### 2. Validation
- Added `az800l04-vm0` as a connection in Windows Admin Center.
- Signed in with the instructor-provided credentials when prompted.
- Successfully connected to the Azure VM.
- Reviewed the Overview pane for the connected VM.

---

### Evidence
> **[Picture]**
> *Capture the Windows Admin Center gateway in Azure, the PowerShell Remoting configuration on az800l04-vm0, and the successful connection to the Azure VM from the gateway.*

---

### Professional Insight
- **Gateway-Based Management:** Running Windows Admin Center in Azure provides a consistent administration experience for cloud-based servers.
- **PowerShell Remoting:** Enabling WinRM and PSRemoting is required for remote management scenarios that rely on PowerShell.
- **Hybrid Visibility:** Using the Azure-hosted gateway to manage Azure VMs demonstrates that WAC can serve as a common management plane across environments.

---

## Exercise 5: Deprovisioning the Azure environment

### Task 1: Start a PowerShell session in Cloud Shell
In this task, I opened a PowerShell session in Azure Cloud Shell from the Azure portal.

#### 1. Cloud Shell Access
- **Portal:** `Azure portal`
- **Shell Type:** `PowerShell`

#### 2. Validation
- Opened Cloud Shell successfully.
- Confirmed that the PowerShell session was available.

---

### Task 2: Identify all Azure resources provisioned in the lab
In this task, I listed and removed the Azure resource groups created during the lab.

#### 1. Resource Group Review
```powershell
Get-AzResourceGroup -Name 'az800l04*'
```

#### 2. Resource Group Deletion
```powershell
Get-AzResourceGroup -Name 'az800l04*' | Remove-AzResourceGroup -Force -AsJob
```

#### 3. Validation
- Confirmed that the matching resource groups were identified.
- Started asynchronous deletion of all matching resource groups.
- Noted that deletion may take a few minutes to complete because the command runs as a background job.

---

### Evidence
> **[Picture]**
> *Capture Cloud Shell showing the resource group query and the deletion command running as a background job.*

---

### Professional Insight
- **Cost Management:** Deprovisioning lab resources is important to avoid unnecessary Azure charges.
- **PowerShell Cleanup:** `Get-AzResourceGroup` and `Remove-AzResourceGroup` provide a simple way to find and remove lab resources.
- **Asynchronous Deletion:** Using `-AsJob` allows cleanup to continue in the background without blocking the session.
