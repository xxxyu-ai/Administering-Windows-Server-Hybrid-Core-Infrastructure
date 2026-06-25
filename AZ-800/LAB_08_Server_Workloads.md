# AZ-800: Administering Windows Server Hybrid Core Infrastructure

## Lab8: Implementing hybrid networking infrastructure

This lab focused on building a hub-and-spoke network topology in Azure, testing VNet peering transitivity, and configuring user-defined routes to force spoke-to-spoke traffic through the hub.

---

## Exercise 1: Implement virtual network routing in Azure

### Task 1: Provision lab infrastructure resources
In this task, I deployed the Azure lab infrastructure using an ARM template and installed the Network Watcher extension on the deployed VMs.

#### 1. Resource Deployment
- **Resource Group:** `AZ800-L0801-RG`
- **Template Files:**
  - `L08-rg_template.json`
  - `L08-rg_template.parameters.json`

#### 2. Validation
- Created the resource group in the selected Azure region.
- Deployed the virtual networks and VMs from the template.
- Installed the Network Watcher extension on all deployed VMs.

---

### Task 2: Configure the hub and spoke network topology
In this task, I created VNet peerings to form a hub-and-spoke topology.

#### 1. Hub VNet
- **Hub:** `az800l08-vnet0`

#### 2. Spoke VNets
- **Spoke 1:** `az800l08-vnet1`
- **Spoke 2:** `az800l08-vnet2`

#### 3. Peering Configuration
- `az800l08-vnet0_to_az800l08-vnet1`
- `az800l08-vnet1_to_az800l08-vnet0`
- `az800l08-vnet0_to_az800l08-vnet2`
- `az800l08-vnet2_to_az800l08-vnet0`

#### 4. Validation
- Confirmed peering was created in both directions.
- Enabled forwarded traffic where required.
- Completed the hub-and-spoke topology.

---

### Task 3: Test transitivity of virtual network peering
In this task, I used Network Watcher to test connectivity between the Azure VMs and confirm peering behavior.

#### 1. Connectivity Tests
- **Source VM:** `az800l08-vm0`
- **Destination 1:** `10.81.0.4`
- **Destination 2:** `10.82.0.4`

#### 2. Validation Results
- Confirmed connectivity to `10.81.0.4` was **Reachable**.
- Confirmed connectivity to `10.82.0.4` was **Reachable**.
- Reviewed the network path and noted that the connections were direct through the hub.
- Confirmed that traffic between the two spoke VNets was **Unreachable** before routing was configured.
- Verified that VNet peering is non-transitive by default.

---

### Task 4: Configure routing in the hub and spoke topology
In this task, I configured the hub VM to function as a router and created UDRs to enable spoke-to-spoke routing.

#### 1. Enable Routing on the Hub VM
- **VM:** `az800l08-vm0`
- Enabled IP forwarding on the VM NIC.
- Installed the Remote Access role.
- Installed the Routing role service.
- Enabled forwarding in the guest OS.

#### 2. Route Table Configuration
- **Route Table 1:** `az800l08-rt12`
- **Route Table 2:** `az800l08-rt21`
- **Next Hop Type:** `Virtual appliance`
- **Next Hop Address:** `10.80.0.4`

#### 3. Route Entries
- `az800l08-route-vnet1-to-vnet2` → `10.82.0.0/20`
- `az800l08-route-vnet2-to-vnet1` → `10.81.0.0/20`

#### 4. Subnet Associations
- Associated `az800l08-rt12` with `az800l08-vnet1/subnet0`
- Associated `az800l08-rt21` with `az800l08-vnet2/subnet0`

#### 5. Validation
- Confirmed that traffic from `az800l08-vm1` to `10.82.0.4` became **Reachable**.
- Reviewed the network path and confirmed traffic was routed via `10.80.0.4`.
- Verified that the hub VM acted as a router between the spoke VNets.

---

## Exercise 2: Implement DNS name resolution in Azure

### Task 1: Create a private DNS zone
In this task, I created and configured an Azure private DNS zone for the lab environment.

#### 1. DNS Zone Details
- **Private DNS Zone:** `az800l08.com`
- **Associated VNets:** Hub and spoke virtual networks

#### 2. Validation
- Created the private DNS zone successfully.
- Linked it to the required virtual networks.

---

### Task 2: Configure external name resolution
In this task, I configured DNS settings so the lab environment could resolve external names.

#### 1. DNS Forwarding Configuration
- Configured Azure DNS or forwarders as required by the lab scenario.

#### 2. Validation
- Confirmed external DNS resolution worked correctly from the deployed VMs.

---

## Exercise 3: Deprovision the Azure environment

### Task 1: Remove lab resources
In this task, I removed the Azure resource groups created for the lab.

#### 1. Cleanup Command
```powershell
Get-AzResourceGroup -Name 'AZ800-L08*'
Get-AzResourceGroup -Name 'AZ800-L08*' | Remove-AzResourceGroup -Force -AsJob
```

#### 2. Validation
- Confirmed the matching resource groups were identified.
- Started asynchronous deletion of the lab resources.
- Noted that resource deletion continues in the background.

---

### Professional Insight
- **Hub-and-Spoke Design:** This topology centralizes routing and security controls.
- **UDR-Based Routing:** User-defined routes let you override default Azure system routes for controlled traffic flow.
- **Network Watcher:** Connection troubleshoot is useful for validating reachability and identifying routing behavior.

---

## Exercise 2: Implement DNS name resolution in Azure

### Task 1: Configure Azure private DNS name resolution
In this task, I created a private DNS zone and linked it to the lab virtual networks.

#### 1. Private DNS Zone
- **Resource Group:** `AZ800-L0802-RG`
- **Zone Name:** `contoso.org`
- **Location:** `<Azure region used in Exercise 1>`

#### 2. Virtual Network Links
- `az800l08-vnet0-link`
- `az800l08-vnet1-link`
- `az800l08-vnet2-link`

#### 3. Validation
- Created the private DNS zone successfully.
- Enabled auto-registration on all three virtual network links.
- Verified that `az800l08-vm0`, `az800l08-vm1`, and `az800l08-vm2` appeared as auto-registered A records in the zone.

---

### Task 2: Validate Azure private DNS name resolution
In this task, I validated that name resolution worked between virtual networks by using the private DNS zone.

#### 1. Connectivity Test
- **Source VM:** `az800l08-vm1`
- **Destination FQDN:** `az800l08-vm2.contoso.org`
- **Protocol:** `TCP`
- **Port:** `3389`

#### 2. Validation
- Verified that the connection status was **Reachable**.
- Confirmed that the FQDN resolved through the Azure private DNS zone.

---

### Task 3: Configure Azure public DNS name resolution
In this task, I created a public DNS zone and added an A record for a simple web name.

#### 1. Public DNS Zone
- **Resource Group:** `AZ800-L0802-RG`
- **Domain Name:** `<domain name identified in GoDaddy search>`

#### 2. Record Set
- **Name:** `www`
- **Type:** `A`
- **Alias Record Set:** `No`
- **TTL:** `1 hour`
- **IP Address:** `20.30.40.50`

#### 3. Name Server
- Recorded the full name of **Name server 1** for later testing.

#### 4. Validation
- Created the public DNS zone successfully.
- Added the `www` A record.
- Recorded the assigned Azure name server.

---

### Task 4: Validate Azure public DNS name resolution
In this task, I used `nslookup` to verify external DNS resolution against the Azure name server.

#### 1. Validation Command
```powershell
nslookup www.<domain name> <Name server 1>
```

#### 2. Validation
- Verified that the output returned the public IP address `20.30.40.50`.
- Confirmed that the Azure public DNS zone was responding correctly when queried directly.

---

### Professional Insight
- **Private DNS Zones:** Azure private DNS zones simplify internal name resolution across linked virtual networks.
- **Auto Registration:** Auto-registration reduces manual record maintenance for Azure VMs.
- **Public DNS Zones:** Azure public DNS zones can be queried directly and are useful for testing authoritative DNS records.

---

## Exercise 3: Deprovisioning the Azure environment

---

### Task 1: Start a PowerShell session in Cloud Shell
In this task, I opened a PowerShell session in Azure Cloud Shell from the Azure portal.

#### 1. Cloud Shell Access
- **Portal:** `Azure portal`
- **Shell Type:** `PowerShell`

#### 2. Validation
- Opened Cloud Shell successfully.
- Accepted the default settings if this was the first launch.

---

### Task 2: Identify all Azure resources provisioned in the lab
In this task, I listed and deleted the resource groups created throughout the lab.

#### 1. List Resource Groups
```powershell
Get-AzResourceGroup -Name 'AZ800-L08*'
```

#### 2. Delete Resource Groups
```powershell
Get-AzResourceGroup -Name 'AZ800-L08*' | Remove-AzResourceGroup -Force -AsJob
```

#### 3. Validation
- Confirmed that all matching resource groups were listed.
- Started asynchronous deletion of the lab resources.
- Noted that deletion continues in the background because of the `-AsJob` parameter.

---

### Professional Insight
- **Cost Control:** Deprovisioning unused lab resources helps avoid unnecessary Azure charges.
- **Efficient Cleanup:** PowerShell makes it easy to remove multiple lab resource groups at once.
- **Asynchronous Deletion:** The `-AsJob` switch allows deletion to continue without blocking the current session.

- 
