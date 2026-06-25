# AZ-800: Administering Windows Server Hybrid Core Infrastructure

## Lab7: Implementing and configuring network infrastructure services in Windows Server

This lab focused on deploying and configuring DHCP with failover, creating a custom DNS server for Trey Research, and preparing Windows Admin Center for remote management.

---

## Exercise 1: Deploying and configuring DHCP

### Task 1: Install the DHCP role
In this task, I installed the DHCP role on SEA-SVR1 and prepared Windows Admin Center for managing DHCP and DNS.

#### 1. Windows Admin Center Setup
- Installed Windows Admin Center on SEA-ADM1 if needed.
- Connected to `https://SEA-ADM1.contoso.com`.
- Installed the **DHCP** and **DNS** extensions in Windows Admin Center.

#### 2. DHCP Role Installation
- Added `sea-svr1.contoso.com` as a connection in Windows Admin Center.
- Used **Roles & features** to install the DHCP role on SEA-SVR1.
- Installed the **DHCP PowerShell** tools from the DHCP tool.

#### 3. Validation
- Confirmed the DHCP and DNS extensions were installed.
- Verified the DHCP role was installed successfully on SEA-SVR1.

---

<img width="1147" height="697" alt="lab7 e1-1" src="https://github.com/user-attachments/assets/d584be36-3092-4045-8d4f-b1536f2410fa" />

---

### Task 2: Authorize the DHCP server
In this task, I completed the DHCP post-install configuration so the server could serve addresses in Active Directory.

#### 1. Authorization
- **DHCP Server:** `SEA-SVR1`
- **Post-Install Wizard:** Completed with default options

#### 2. Validation
- Opened Server Manager notifications.
- Completed DHCP configuration successfully.
- Confirmed the server was authorized.

---

<img width="1026" height="820" alt="lab7 e1-2" src="https://github.com/user-attachments/assets/39e5d49e-7170-4ab6-b381-b12c840e6eb3" />

---

### Task 3: Create a scope
In this task, I created the ContosoClients IPv4 scope and configured the DNS server option.

#### 1. Scope Configuration
- **Protocol:** `IPv4`
- **Scope Name:** `ContosoClients`
- **Starting IP Address:** `10.100.150.50`
- **Ending IP Address:** `10.100.150.254`
- **Subnet Mask:** `255.255.255.0`
- **Router:** `10.100.150.1`
- **Lease Duration:** `4 days`

#### 2. Scope Option
- **Option 006 DNS Servers:** `172.16.10.10`

#### 3. Validation
- Created the scope successfully.
- Added the DNS server scope option.
- Added authorized servers in the DHCP console as required.

---

<img width="1117" height="813" alt="lab7 e1-3" src="https://github.com/user-attachments/assets/2f7a9fbd-f6e7-4e0e-ad93-e4b319057563" />

---

### Task 4: Configure DHCP Failover
In this task, I configured failover between SEA-SVR1 and SEA-DC1 for high availability.

#### 1. Failover Relationship
- **Relationship Name:** `SEA-SVR1 to SEA-DC1`
- **Maximum Client Lead Time:** `1 hour`
- **Mode:** `Hot standby`
- **Partner Role:** `Standby`
- **Standby Reservation:** `5%`
- **State Switchover Interval:** `Disabled`
- **Message Authentication:** `Enabled`
- **Shared Secret:** `DHCP-Failover`

#### 2. Validation
- Verified that SEA-SVR1 had one scope before the second relationship was configured.
- Verified that SEA-DC1 had two scopes after failover was added.
- Reused the existing failover relationship when configuring the Contoso scope on SEA-DC1.
- Confirmed both scopes appeared under SEA-SVR1 after configuration.

---

### Task 5: Verify DHCP functionality
In this task, I changed the client configuration to DHCP and verified failover by switching the active DHCP server.

#### 1. Client Lease Validation
- Changed the network adapter from static to automatic addressing.
- Verified that SEA-ADM1 obtained a DHCP lease from `172.16.10.12`.

#### 2. Failover Verification
- Confirmed both DHCP servers listed the lease for SEA-ADM1 in the Contoso scope.
- Stopped the DHCP service on SEA-SVR1.
- Disabled and re-enabled the network adapter to force renewal.
- Confirmed the lease was obtained from `SEA-DC1` (`172.16.10.10`).

#### 3. Validation
- Verified DHCP failover worked correctly.
- Confirmed the client remained reachable after the primary server stopped responding.

---

### Professional Insight
- **High Availability:** DHCP failover provides continuity when one DHCP server becomes unavailable.
- **Administrative Efficiency:** Windows Admin Center and the DHCP console make DHCP provisioning and management easier.
- **Network Resilience:** Testing failover ensures that clients can continue to receive leases during service disruption.

---

## Exercise 2: Deploying and configuring DNS

### Task 1: Install the DNS role
In this task, I installed the DNS role on SEA-SVR1 and prepared the DNS management tools in Windows Admin Center.

#### 1. DNS Role Installation
- **Target Server:** `SEA-SVR1`
- **Tools Used:** `Windows Admin Center`

#### 2. Validation
- Installed the DNS role using **Roles & features**.
- Installed the **DNS PowerShell** tools from the DNS tool.
- Confirmed the DNS extension was available in Windows Admin Center.

---

<img width="1146" height="647" alt="lab7 e2-1" src="https://github.com/user-attachments/assets/ad1bd783-81d9-4cbf-9715-3d803bdfed6b" />

---

### Task 2: Create a DNS zone
In this task, I created a new primary DNS zone and added a host record for the test application.

#### 1. DNS Zone Configuration
- **Zone Type:** `Primary`
- **Zone Name:** `TreyResearch.net`
- **Zone File:** `Create a new file`
- **Zone File Name:** `TreyResearch.net.dns`
- **Dynamic Update:** `Do not allow dynamic update`

#### 2. Host Record Configuration
- **Record Type:** `Host (A)`
- **Record Name:** `TestApp`
- **IP Address:** `172.30.99.234`
- **TTL:** `600`

#### 3. Validation
```powershell
Resolve-DnsName -Server sea-svr1.contoso.com -Name testapp.treyresearch.net
```
- Verified that the new record resolved correctly.

---

### Task 3: Configure forwarding
In this task, I configured DNS forwarding so Trey Research DNS could resolve Internet names.

#### 1. Forwarder Configuration
- **DNS Server:** `SEA-SVR1.contoso.com`
- **Forwarder Address:** `131.107.0.100`

#### 2. Validation
- Opened DNS Manager from Server Manager.
- Configured the forwarder on the server properties page.
- Confirmed the forwarder was added successfully.

---

### Task 4: Configure conditional forwarding
In this task, I created a conditional forwarder for Contoso.com so Trey Research DNS could resolve internal Contoso names.

#### 1. Conditional Forwarder Configuration
- **Domain:** `Contoso.com`
- **Forward To:** `SEA-DC1.contoso.com (172.16.10.10)`

#### 2. Validation
```powershell
Resolve-DnsName -Server sea-svr1.contoso.com -Name sea-dc1.contoso.com
```
- Confirmed the conditional forwarder worked correctly.

---

### Task 5: Configure DNS policies
In this task, I configured DNS policies so that `testapp.treyresearch.net` would resolve differently based on client location.

#### 1. Client Subnet
```powershell
Add-DnsServerClientSubnet -Name "HeadOfficeSubnet" -IPv4Subnet "172.16.10.0/24"
```

#### 2. Zone Scope
```powershell
Add-DnsServerZoneScope -ZoneName "TreyResearch.net" -Name "HeadOfficeScope"
```

#### 3. Resource Record in Zone Scope
```powershell
Add-DnsServerResourceRecord -ZoneName "TreyResearch.net" -A -Name "testapp" -IPv4Address "172.30.99.100" -ZoneScope "HeadOfficeScope"
```

#### 4. Query Resolution Policy
```powershell
Add-DnsServerQueryResolutionPolicy -Name "HeadOfficePolicy" -Action ALLOW -ClientSubnet "eq,HeadOfficeSubnet" -ZoneScope "HeadOfficeScope,1" -ZoneName "TreyResearch.net"
```

#### 5. Validation
- Confirmed the policy was created successfully.
- Verified the head office subnet was mapped to the head office zone scope.

---

### Task 6: Verify DNS policy functionality
In this task, I verified that DNS resolution changed based on whether the client was inside or outside the head office subnet.

#### 1. Head Office Verification
- Ran `ipconfig` and confirmed SEA-ADM1 was on `172.16.10.0/24`.
- Ran DNS resolution and verified `testapp.treyresearch.net` resolved to `172.30.99.100`.

#### 2. Outside Subnet Verification
- Changed the IP configuration of SEA-ADM1 to `172.16.11.11`.
- Used subnet mask `255.255.0.0`.
- Set DNS server IP to `172.60.10.12`.
- Ran DNS resolution again and verified `testapp.treyresearch.net` resolved to `172.30.99.234`.

#### 3. Validation
- Confirmed the DNS policy returned the head office IP for head office clients.
- Confirmed clients outside the subnet received the alternate IP address.
- Restored the original IP configuration of SEA-ADM1.

---

### Professional Insight
- **DNS Forwarding:** Forwarding keeps Internet name resolution working while preserving internal DNS control.
- **Conditional Forwarding:** Conditional forwarders are useful for directing specific domain queries to the correct internal DNS server.
- **DNS Policies:** DNS policies allow location-based responses, which is useful for testing, traffic steering, and environment-specific name resolution.
