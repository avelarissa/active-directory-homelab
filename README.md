# active-directory-homelab
Homelab de Active Directory simulando um ambiente corporativo utilizando Windows Server 2022, incluindo implantação de Domain Controller, configuração de DNS e integração de cliente ao domínio.

# Active Directory Home Lab: Domain Deployment and Management (Windows Server 2022)

## Project Description

This repository presents the documentation regarding the configuration of an Active Directory (AD) environment in a controlled and isolated laboratory setting, with the purpose of understanding the fundamental concepts of domain management, centralized authentication, and Group Policy Objects (GPO).

The laboratory consisted of a Domain Controller (Windows Server 2022) and a client (Windows 10 Pro), interconnected in an isolated network configured through VirtualBox. The following activities were performed: installation and configuration of AD DS and DNS, creation of domain users, configuration of Remote Desktop access, joining the client to the domain, and application of password policies via Group Policy.

The documentation includes environment setup, commands used, evidence of the tests (screenshots), as well as analyses and best practices for Active Directory domain administration.

>⚠️ Disclaimer: All procedures, commands, and configurations presented in this repository were performed in an isolated virtual lab environment and are intended solely for educational and training purposes. The configurations demonstrated are simplified examples and should not be considered production-ready or a substitute for proper security assessments in real-world environments.

## Tools and Technologies Used

Below are the main technologies and tools employed in the laboratory.

| Tool / Technology                     | Main Function        | Use in the Laboratory |
|--------------------------------------|---------------------|----------------------|
| VirtualBox                           | Virtualization      | Virtualization platform used to create and manage the lab VMs (Windows Server 2022 and Windows 10). Supports internal networks and snapshots. |
| Windows Server 2022                  | Domain Controller   | Server operating system hosting Active Directory Domain Services (AD DS) and DNS. |
| Windows 10 Pro                       | Domain Client       | Workstation that joins the domain and authenticates using Active Directory users. |
| Active Directory Domain Services (AD DS) | Directory Service   | Centrally manages users, computers, and security policies. |
| DNS Server                           | Name Resolution     | Allows clients to locate the Domain Controller through the domain name. |
| Group Policy Management (GPMC)       | Policy Management   | Applies security configurations (e.g., password policy) to domain users and computers. |
| Remote Desktop Protocol (RDP)        | Remote Access       | Allows the administrator to manage the server and domain users to access the client remotely. |

## Official Download Links

| Tool / Technology   | Version | Link |
|--------------------|--------|------|
| VirtualBox         | 7.0+   | [virtualbox.org](https://www.virtualbox.org/)|
| Windows Server 2022 | 2022  | [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)|
| Windows 10 Pro     | 22H2   | [Microsoft Software Download](https://www.microsoft.com/en-us/software-download/windows10)|

> Note: The evaluations of Windows Server 2022 and Windows 10 Pro are free and work for 180 days, which is sufficient for educational purposes.

## Environment Setup

### Installation of VirtualBox, Creation of VMs (Windows Server 2022 and Windows 10), and Internal Network Preparation

This section presents the procedures and parameters adopted for configuring the virtual environment used in the laboratory.

Below are reproducible instructions that allow creating a laboratory similar to the one used in this project, with Windows Server 2022 (Domain Controller), Windows 10 Pro (client), and a dedicated internal network.

### A. Install VirtualBox

1. Access the official VirtualBox website.
2. Download the installer appropriate for your system (Windows / macOS / Linux).
3. Run the installer by double-clicking the downloaded file. Then, follow the wizard instructions and accept the installation of network drivers when prompted.
4. Finish and, if prompted, restart your computer.
5. Open VirtualBox and confirm the interface.

<div align="right"> <details> <summary font-weight: bold;> [VirtualBox Interface] </summary> <img src="images/01-virtualbox-interface.png" alt="VirtualBox initial interface" width="600"> </details> </div>

### B. Create the Windows Server 2022 VM

1. Access the Microsoft website and download the Windows Server 2022 ISO (180-day evaluation).
2. In VirtualBox, click New.
3. VM Name: LAB-DC01 → Folder: select your desired folder → ISO Image: click the dropdown and select Choose a disk file... → browse and select the Windows Server 2022 ISO you downloaded → Next →

<div align="right"> <details> <summary font-weight: bold;> [Windows Server 2022 VM Configuration] </summary> <img src="images/02-winserver-config.png" alt="Windows Server 2022 VM creation screen with ISO" width="600"> </details> </div>

4. Unattended Guest OS Setup: UNCHECK "Proceed with Unattended Installation" (this disables the buggy auto-setup) → Next →
5. Hardware:
- Base Memory: 4096 MB (2 GB)
- Processors: 2

<div align="right"> <details> <summary font-weight: bold;> [Windows Server VM Hardware] </summary> <img src="images/03-winserver-hardware.png" alt="Virtual hardware specifications" width="600"> </details> </div>

6. Virtual Hard Disk:
- Disk Size: 50 GB
7. Click Finish to create the VM. The installation will start automatically.

### C. Create the Windows 10 Pro VM

1. Download the Windows 10 Pro ISO (Pro edition is required to join the domain).
2. In VirtualBox, click New.
3. VM Name: CLIENT01 → Folder: select your desired folder → ISO Image: click the dropdown and select Choose a disk file... → browse and select the Windows 10 ISO you downloaded → Next →

<div align="right"> <details> <summary font-weight: bold;> [Windows 10 VM Configuration] </summary> <img src="images/05-win10-config.png" alt="Windows 10 VM creation screen" width="600"> </details> </div>

4. Unattended Guest OS Setup: UNCHECK "Proceed with Unattended Installation" → Next →
5. Hardware:
- Base Memory: 4096 MB (4 GB)
- Processors: 2 → Next →
6. Virtual Hard Disk:
- Disk Size: 50 GB → Finish
- Click Finish to create the VM.

### D. Internal Network Configuration

1. For each VM (Windows Server 2022 and Windows 10):
2. Select the VM → Settings → Network.
3. Under Adapter 1, make sure Enable Network Adapter is checked.
4. Under Attached to, choose Internal Network.
5. In the Name field, type AD-Lab-Network (use the same name for both VMs).

<div align="right"> <details> <summary font-weight: bold;> [Internal Network Configuration] </summary> <img src="images/06-network-internal.png" alt="Internal Network adapter configuration" width="600"> </details> </div>

6. Save the settings.

> Why Internal Network? This configuration completely isolates the laboratory from your home network and the internet, ensuring a controlled, secure, and predictable environment for learning.

### E. Complete the OS Installation (if needed)
Since you attached the ISO during VM creation, the installation will begin automatically when you start each VM.

Windows Server 2022
1. Select the DC01 VM and click Start.
2. The Windows Server 2022 installer will load. Follow the prompts.
3. When asked, choose Desktop Experience (not Core).
4. Set the Administrator password when prompted.

Windows 10 Pro
1. Select the CLIENT01 VM and click Start.
2. The Windows 10 installer will load. Follow the prompts.

> ⚠️ Important: You MUST select Windows 10 Pro during installation. The Pro edition is required to join a domain. Windows 10 Home does NOT support domain membership.

### F. Configure IP Addresses
When you set the network adapter to Internal Network and start the VM for the first time, Windows will try to find a DHCP server to automatically assign an IP address. Since there is no DHCP server on an Internal Network, Windows will eventually give up and assign itself an APIPA address.

You can see this by opening Command Prompt and typing ipconfig. You will likely see an IP address in the range 169.254.x.x (for example, 169.254.101.98).

<div align="right"> <details> <summary font-weight: bold;> [APIPA Address on Windows Server] </summary> <img src="images/xx-apipa-address.png" alt="APIPA address shown in ipconfig command output" width="600"> </details> </div>

> What is APIPA? APIPA stands for Automatic Private IP Addressing. It is a feature in Windows that automatically assigns an IP address from the range 169.254.0.1 to 169.254.255.254 when no DHCP server is available. APIPA allows computers on the same network segment to communicate with each other without any manual configuration. However, APIPA addresses are unpredictable and can change after a reboot. For a lab environment where our server needs to be found at a consistent address by the client, this is a problem.
