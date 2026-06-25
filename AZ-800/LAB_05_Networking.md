# AZ-800: Administering Windows Server Hybrid Core Infrastructure

## Lab5: Implementing and configuring virtualization in Windows Server

This lab focused on creating and configuring a Hyper-V virtual machine, managing virtual switches and disks, and using Windows Admin Center to administer virtual machines and Hyper-V host settings.

---

## Exercise 1: Creating and configuring VMs

### Task 1: Create a Hyper-V virtual switch
In this task, I created a private virtual switch on SEA-SVR1 by using Hyper-V Manager.

#### 1. Switch Configuration
- **Switch Name:** `Contoso Private Switch`
- **Connection Type:** `Private network`
- **Target Host:** `SEA-SVR1`

#### 2. Validation
- Opened Hyper-V Manager from Server Manager.
- Created the private virtual switch successfully.
- Confirmed the switch appeared in Virtual Switch Manager.

---

### Task 2: Create a virtual hard disk
In this task, I created a differencing virtual hard disk for SEA-VM1.

#### 1. Disk Configuration
- **Disk Format:** `VHD`
- **Disk Type:** `Differencing`
- **Disk Name:** `SEA-VM1`
- **Location:** `C:\Base`
- **Parent Disk:** `C:\Base\BaseImage.vhd`

#### 2. Validation
- Confirmed the differencing VHD was created successfully.
- Verified the disk was stored in the correct location.

---

### Task 3: Create a virtual machine
In this task, I created a Generation 1 virtual machine that used the private switch and the differencing disk.

#### 1. VM Configuration
- **VM Name:** `SEA-VM1`
- **Location:** `C:\Base`
- **Generation:** `Generation 1`
- **Memory:** `4096 MB`
- **Virtual Switch:** `Contoso Private Switch`
- **Virtual Disk:** `C:\Base\SEA-VM1.vhd`

#### 2. Additional Configuration
- Enabled **Dynamic Memory**.
- Set **Maximum RAM** to `4096 MB`.

#### 3. Validation
- Confirmed the VM was created successfully.
- Verified the VM settings reflected the required configuration.

---

### Task 4: Manage Virtual Machines using Windows Admin Center
In this task, I used Windows Admin Center to manage SEA-VM1 and review Hyper-V host resources.

#### 1. Windows Admin Center Setup
- **WAC URL:** `https://SEA-ADM1.contoso.com`
- **Managed Host:** `sea-svr1.contoso.com`

#### 2. VM Management Actions
- Opened the **Virtual Machines** tool.
- Reviewed the **Summary** pane.
- Opened `SEA-VM1` in the Inventory pane.
- Reviewed the VM settings.
- Created a new 5 GB disk.
- Started `SEA-VM1` and reviewed the running statistics.
- Shut down `SEA-VM1`.

#### 3. Virtual Switch Review
- Opened the **Virtual switches** tool.
- Identified the available virtual switches on the host.

#### 4. Validation
- Confirmed Windows Admin Center could manage the Hyper-V host and VM.
- Verified the VM could be started, monitored, and shut down from WAC.

---

### Professional Insight
- **Hyper-V Design:** Using a private switch isolates lab workloads from external network traffic.
- **Differencing Disks:** Differencing VHDs save space and make lab VM deployment faster when a base image already exists.
- **Windows Admin Center:** WAC provides a consistent interface for managing Hyper-V hosts and VMs without opening Hyper-V Manager directly.

---

## Exercise 2: Installing and configuring containers

### Task 1: Install Docker on Windows Server
In this task, I installed Docker CE on SEA-SVR1 by using a PowerShell Remoting session.

#### 1. Docker Installation
```powershell
Invoke-WebRequest -UseBasicParsing "https://raw.githubusercontent.com/microsoft/Windows-Containers/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1" -o install-docker-ce.ps1
.\install-docker-ce.ps1
Restart-Computer -Force
```

#### 2. Validation
- Confirmed Docker CE was downloaded and installed successfully.
- Restarted SEA-SVR1 after installation.
- Reconnected to SEA-SVR1 after the restart.

---

### Task 2: Install and run a Windows container
In this task, I downloaded a Nano Server image, launched a container, created a file inside the container, and then committed the changes into a custom image.

#### 1. Verify Local Images
```powershell
docker images
```
- Confirmed that no images were present initially.

#### 2. Download the Nano Server Image
```powershell
docker pull mcr.microsoft.com/windows/nanoserver:ltsc2022
docker images
```

#### 3. Run the Container
```powershell
docker run -it mcr.microsoft.com/windows/nanoserver:ltsc2022 cmd.exe
hostname
echo "Hello World!" > C:\Users\Public\Hello.txt
exit
```

#### 4. Identify the Container and Commit Changes
```powershell
docker ps -a
docker commit <containerID> helloworld
docker images
```

#### 5. Validate the Custom Image
```powershell
docker run --rm helloworld cmd.exe /s /c type C:\Users\Public\Hello.txt
docker run --rm mcr.microsoft.com/windows/nanoserver:ltsc2022 cmd.exe /s /c type C:\Users\Public\Hello.txt
```

#### 6. Validation
- Confirmed the Nano Server image was downloaded successfully.
- Confirmed the container could run interactively.
- Verified that `Hello.txt` was created inside the container.
- Created a custom image named `helloworld`.
- Confirmed the custom image contained the file.
- Verified that the original Nano Server image did not include the file.

---

### Task 3: Use Windows Admin Center to manage containers
In this task, I used Windows Admin Center to manage Docker containers on SEA-SVR1.

#### 1. Extension Verification
- Opened the **Extensions** pane in Windows Admin Center.
- Verified the **Containers** extension was installed and updated.

#### 2. Container Management
- Opened the **Containers** tool from the SEA-SVR1 management view.
- Continued after being prompted to close the PowerShell session.
- Reviewed the **Overview**, **Containers**, **Images**, **Networks**, and **Volumes** tabs.

#### 3. Validation
- Confirmed the Containers extension was available in Windows Admin Center.
- Verified that container management views were accessible from the host.

---

### Professional Insight
- **Container Fundamentals:** Containers share the host kernel, so they are lighter than virtual machines.
- **Image Immutability:** Committing a running container creates a custom image that preserves file changes.
- **Windows Admin Center:** WAC provides a convenient graphical interface for viewing container state, images, networks, and volumes.
