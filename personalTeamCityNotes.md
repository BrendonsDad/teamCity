# Team City
Team city is a CD CI deployment tool that helps deploy code and projects.

### Setup CI/CD Tool and Configuring Builds
Traditionally CI/CD tools need a lot of administration. These requires a lot of plugins.
The main advantage of Team City is that the set up and administration is way easier.


### Configuration and Code
You can script the builds in Team city with KOTLIN an open source language.


### Build Chains, also called Pipeline
Sequence of connected dependencies
run builds in parallel.

### Personal Builds
You can run personal builds which are available for all major IDEs.

## How does it work?
Centralized
Team city has build agents you can add and connect

1. Install Team City Server
2. Configure CI Pipeline
3. Connect Build Agents
4. Run our build


### Steps break down
(check out this link for a break down)
https://youtu.be/zqi4fDF-S60?si=MsOkbreng80X6PYO&t=450

1. Install Team City Server
    * This is the main component where you can create your builds
    * You can install teamCity online
    * Possibility to use teamcity cloud, but docker is more manual
    * There are some settings to configure and things to download from the terminal, watch the video for that.

2. Configure CI Pipeline
    * Connect your repository 

3. Deploy to Agents
    * We need to connect an agent to our team city server






///////////////////////////////////////////////////////////////////////////////////////////////////////////
///////////////////////////////////////////////////////////////////////////////////////////////////////////

# TeamCity Agent Setup (Windows 11)

///////////////////////////////////////////////////////////////////////////////////////////////////////////
///////////////////////////////////////////////////////////////////////////////////////////////////////////


## VM SetUp

### ChatGPT overview of this Section
#### What is TeamCity
TeamCity is a Continuous Integration (CI) and Continuous Deployment (CD) tool developed by JetBrains. It's used to automate the building, testing, and deployment of software projects.

For your game project, TeamCity will help you:
* Automate your build process (e.g., compiling the game)
* Run tests automatically
* Deploy updates efficiently

#### What is a VM (Virtual Machine)?
A Vitrual Machine (VM) is a simulated computer system that runs inside another system. It behaves like a physical computer but is actually a software-based enviornment. VMs are useful because they allow you to run multiple operating systems on the same physical machine, making them ideal for testing, development, and server hosting.

In this case, you're setting up a VM using Proxmox, which is a virtualization platform. (watch a video on proxmox)

#### Breaking Down the Documentation
The documentation provides instructions to set up a Windows 11 VM on proxmox. Let's analyze it step by step.
1. Resources:
    * You need to download:
        * Windows 11 ISO (operating system installation file)
        * VirtIO drivers (drivers that help)
2. Upload images to Proxmox
    * After downloading, you upload these files to Proxmox so they can be used to create a VM. 
3. VM Creation:
    * Name: Call it "windows."
    * OS Settings:
        * Select the Windows 11 ISO file for installation..
        * Specify that the OS is Windows 11/2022/2025
        * Attach the VirtIO drivers ISO to the VM (this ensures the VM has necessary drivers).
    * System Settings:
        * Qemu Agent: Helps the VM interact better with the Proxmox hypervisor.
        * EFI Storage / TPM Storage: Used for secure boot and encryption support.
    * Diks Settings:
        * Allocate storage (90% of the host machine's storae is recommended)
    * CPU & Memory:
        * Assigne CPU cores and memory to the VM (match the host machine but leave 1GB free for Proxmox).
    * Final Steps:
        * After creation, go to VM Options and enable "Start at boot" so the VM automatically starts when the host machine does.

### Connors notes
* Create a new VM called on the desired notde with the following options:
    * General/Name: Windows
    * OS/Iso Image: (Windows 11 iso uploaded earlier)
    * OS/Type: Windows
    * OS/Version: 11/2022/2025
    * OS/Add additional drive for VirtIO drivers: true
    * OS/VirtIO ISO: (VirtIO drivers iso uploaded earlier)
    * System/Qemu Agent: true
    * System/EFI Storage: local-lvm
    * System/TPM Storage: local-lvm
    * Disk/Storage: local-lvm
    * Disk/Disk size: (The largest you can make it, typically 90% of the total storage of the host)
    * CPU/Cores: (Same as the host machine)
    * CPU/Type: host
    * Memory/Memory: (Same as teh host machine, minus 1GB to leave room for the hypervisor)
Once the machine is created, go to the Options page for the VM and set "Start at boot" to true.

### What I need to Do?
* Watch a proxmox video
* Talk to Jen about how she pushed content to team city
* Meet with Matthew to deside where to download the VM and what settings to give it

### What Matthew Needs to do
* Meet with me and help me with the software side. 

## Windows Install
If you are using Windows Desktop rahter than a windows server, there are some extra steps to bypass all the OOB (out of box) experience stuff. YOU DO NOT WANT A MICROSOFT ACCOUNT ON THIS MACHINE.

Make sure to install "Windows 11 Pro" or Windows 11 Enterprise", NOT "Windows 11 Home".

When it looks for a disk to install windows on, it will fail, becuase the VirtIO drivers are not installed. Click "Load Driver" and navigate to the VirtIO CD drive and install the drivers. It should be something like "D:\amd64\w11". Once you select that folder the driver "Red Hat VirtIO SCSI pass-through controller" should appear. Install that driver and then click "Install".

### Chat GPT overview
You're installing Windows 11 inside a Virtual Machine (VM), but since its not a physical computer, it doesn't recognize the hard drive at first. This seciton explains how to bypass microsfts setup requirments and install the necesary drivers so windows can detect the hard drive. 

#### Step-by-Step Explanation
1. Choose the Right Windows Version
    * You need Windows 11 Pro or Enterprise, Not Windows 11 Home. 
    * Why? Windows 11 Home forces you to sign in with a Microsoft account,  which you dont want for this vim
2. Bypass Microsoft Account Requirement
    * During setup, Windows will ask you to sign in with a Microsoft account. 
    * This documentation warns you to avoid using a Microsoft account for this VM.
    * Solution: There are tricks to skip this, like disconnecting the internet or using a fake email.
3. Fixing the "No Disk Found" Issue
    * Windows does not see the hard drive at first becuse the VM needs special drivers.
    * Solution: Install VirtIO drivers (these help Windows talk to the vitural hard drive).
4. How to Instal VirtIO Drivers
    * When Windows asks where to install, you won't see any drivers listed.
    * Click "Load Driver" (this lets you manually install missing drivers).
    * Find the VirtIO CD drive (usually labeled as D:).
    * Navigate to D:amd64\w11 (this is where the drivers are stored).
    * You shou see a driver named "Red Hat VirtIO SCSI pass-through controller."
    * Install that driver, then continue with the windows instalation.

Why is this important? skipping the microsoft account keeps your vm independent and easy to manage. Installing virtio drivers allows windows to recognize the vitual hard drive so it can install properly. 

#### What do I need to do?
* Again, get with Matthew about this and just start going through

#### What does matthew need to do?
* The biggest thing I need from matthew is for him to help me set up a server on a machine in the talmaged, and I need him to be there to help me set up passwords.


## Windows Setup
It may fail to connect to a network becuase it needs the network driver. Similar to the disk driver, click "Install driver" and navigate to the VirtIO CD drive and install the network driver. It will most likely make you reboot after it finds the driver. 

When it comes time to setup the user, we only plan on using the Administrator account so the account we create using the OOB experience will be deleted. So it doesnt really matter how you create it. If it forces you to create a user linked to a Microsoft account, you will need to bypass this.

1. Press "Shift+F10" to open a command prompt
2. Run: OOBE\BYPASSNRO
3. Screw Microsoft.

The placeholder user we create will be called "Steve" for historical reasons. It will be deleted later, and all the scripts willl assume this user to be Steve. Just fill out all the security questions with "steve" to get it over with. Turn off ALL telemetry and data collection. You don't want microsoft knowing what you're doing.


### ChatGPT Overview:
* Fixing the network issue
    * Why does this happen? Windows cant find the network (internet) inside the Virtual Machine becuase it doesn't have the right drivers installed yet
    * Just like the hard drive issue, we need to manually install the network drivers.

* How to fid it
    1. during insttallation, when windows fails to connect to a network, click "Install driver" (just like you did for the hard drive).
    2. Go to the VirtIO CD drive (probably D: again)
    3. Look for the network driver folder and install it. 
    4. After installing the driver, Windows may ask you to restart-go ahead and do that.

#### Creating a Temporary User (and Why it's Temporary)
Why do we need a temporary user? 
* During instalation, windows forces you to set up a user account
* But for this setup, we only want to us the build in "Administrator" account later.
* So we create a placehoder user that we will delete later

#### How to Create a temporary User
* You'll be asked to set up a user account (a name, password, security questions, etc.).
* The documentation suggests naming the temporary user "Steve" (this name is just a convention for scripts to work later.)
* Security questions? Just answer "Steve" for all of them--it doesn't matter.

#### Bypassing the Microsoft Account Requirement
Why do we need to bypass it?
* Microsoft really wants you to use a microsoft account when setting up windows. 
* this is annoying becuase we just want a local account (not tied to microsoft).

How to Bypass the Microsoft Account Requirement
1. Press Shift + f10
    * This opens the command prompt (a text-based way to give commands to windows).
2. Type the following command and press Enter:

    OOBE\BYPASSNRO

    * This tells windows to restart the setup process but without forcing an online account.
    * After the restart, you should have an option to create a local user instead of a Microsoft account.
3. Continue Setup Normally
    * Finish creating the temporary "Steve" Account.
    * Disable all telemetry and data collection (this prevents Microsoft from collecting you data usage)

#### Final Thoughts
* The Steve account is just a placeholder - you will delete it later
* The real user will be the built-in Administrator account. 
* The bypass trick lets you skip the forced microsoft login and create a local account instead




#### What I need to do
* Install the network driver
* Make the administrator account. Bypass creating this user linked to a microsoft account by running the commands.
* Make Steve
#### What Matthew to do
* work with me while I do this and provide any administrative passwords for me

## Post-Install

Enable the Administrator account:

* Open a command prompt as Administrator
* Run: net user Adminitrator /active:yes
* Run: net user Aministrator * (Set a password)
* Run: net user Steve /delete
* Reboot and login with the Administrator account

Set the host name using PowerShell:

* Rename-Computer -NewName "<HOST>-win -Restart
* (if the host name is "warden", the computer name should be "wardern-win)
* (so run Rename-Computer -NewName "warden-win" -Restart )

Set timezone to mountain time (if not already set) tzutil /s "Mountain Standard Time"

#### Install remaining VirtIO drivers
* msiexec /i D:\virtio-win-gt-x64.msi /passive
* D:\virtio-win-guest-tools.exe /passive
* Restart (you can also unmount the VirtIO CD drive in Proxmox)

#### Enable RDP
* Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
* (Alternatively, you can use the GUI by searching for "Remote Desktop" in the settings)
* Restart

Going forward you can use RDP to connnect to the machine. This makes copying and pasting much easier. If the session disconnects due to "encryption failure" check out this article (https://superuser.com/questions/1747488)

FROM THIS POINT ON USER POWERSHELL INSTEAD OF CMD!!!

#### Configure automatic login:
* reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoAdminLogon /t Reg_SZ /d 1 /f
* reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v  DefaultUserName /t REG_SZ /d Administrator /f
* reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword /t REG_SZ /d <YourPasswordHere> /f

#### De-bloat Windows (optional, but recommended)
* Open Powershell
* type: irm "https://christitus.com/win" | iex
* Configure the "Tweaks" to your liking

#### Remove WinRE
* reagntc /disable (Check status with reagntc /info)
* Get-Partition | Where-Object -FilterScript {$_.Type -eq 'Recovery'} | Remove-Partition -PassThru -Confirm:$false (Optionally: If that didn't find anything, try removing unkown partisions)
* Get-Partition | Where-Object -FilterScript {$_.Type -eq 'Unknown'} | Remove-Partition -PassThru -Confirm:$false

#### Install Updates
* Add0WindowsCapability -Online -Name UpdateServices-Core
* Install-Module -Name PSWindowsUpdate -Force
* Import-Module PSWindowsUpdate
* Get-WindowsUpdate -Install -Verbose -AcceptAll -AutoReboot
* (NOTE: Update KB5034439, if applicable, may fail to install, this is fine.)
* This process can take a while.

#### Allow PowerShell Scripting
* Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser

#### Enable Long Paths (Important!)
* Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem' -Name "LongPathsEnabled" -Value 1
* New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD - Force

#### Disable Visual Effects
* Set-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects' -Name "VisualFXSetting" -Value 2 

#### Misc Environment Variables
* [System.Environment]::SetEnvironmentVariable('DOTNET_CLI_TELEMETRY_OPTOUT', 'true', [System.EnvironmentVariableTarget]::Machine)

#### Remove Install Media
* In Proxmox, go to the Hardware tab for the VM and remove the two CD drives. (Window11 and VirtIO)

#### Setup CrowdStrike
* We don't use this at Two Giants, but if your IT policy reqires it, do it now.
* At BYU, the CSRs wouldn't whitelist the MAC address of the VM until Falcon was installed. 

### ChatGPT Explanation

#### 1 Enable the Built-in Administrator Account
By default, Windows has a hidden "Administrator" account. Now that the temporary "Steve account is no longer needed, we enable the real Administrator account and remove Steve.

Steps:
1. Open Command Prompt as Administrator (right-click on Command Prompt and choose "Run as Administrator").
2. Run the following commands
    * enable administrator account
        net user Administrator /active:yes
    * Set a Password:
        net user Administrator *
        * This will prompt you to type a new password.
    * Elete the temporary Steve Account:
        net user Steve /delete
3. Restart the computer and log in using the Administrator account

#### 2 Rename the Computer
Each VM should have a unique name to make it easier to manage.

Steps: 
1. Open PowerShell (search for "PowerShell" and run it as Administrator).
2. Run:
    Rename-Computer -NewName "<HOST>-win" -Restart
    * Replace <HOST> with your actual hostname.
    * Example: If your Proxmox host is called "warden", run:
    Rename-Computer -NewName "warden-win" -Restart
3. Your VM will restart with the new name.

#### 3 Set the Time Zone
To avoid incorrect timestamps, set the correct time zone.

Steps: 
1. Open PowerShell as Administrator.
2. Run:
    tzutil /s "Mountain Standard Time"
    * If youre in a different time zone, just replace that.

#### Install the Remaining VirtIO Drivers
These drivers improve the VM's performance and compatibility

Steps:
1. Opent PowerShell as Administrator.
2. Run the following commands:
    msiexec /i D:\virtio-win-gt-x64.msi /passive
    D:\virtio-win-guest-tools.exe /passive

3. Restart the VM. 
4. (Optional) In Proxmox, go to the Hardware tab of the VM and remove the VirtIO Cd drive (since it's no longer needed).


#### 5 Enable Remote Desktop (RDP)
RDP allows you to connect to this VM from another computer

Steps:
1. Open PowerShell as Administrator
2. Run:
    Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
3. Restart VM
4. Now, you can connect using Remote Desktop (RDP).

#### 6 Enable Automatic Login (Optional)
This makes Windows log in automatically to the Administrator account.

Steps: 
1. Open PowerShell as Administrator
2. Run the following Commands (replace <YourPasswordHere> with the actual password):

    reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoAdminLogon /t REG_SZ /d 1 /f
    reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName /t REG_SZ /d Administrator /f
    reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword /t REG_SZ /d <YourPasswordHere> /f

3. Restart the VM to apply the changes.


#### 7 De-Bloat Windows (Optional)
Windows comes with a lot of unnecessary software. This step removes it.

Steps:
1. Open PowerShell as Administrator.
2. Run:
    irm "https://christitus.com/win" | iex
3. Follow the on-screen prompts and disable unnecessary features. 

#### 8 Remove Windows Recovery (WinRE)
This step frees up space by removing the Windows Recovery Partition.

Steps:
1. Open PowerShell as administrator
2. Disable Windows Recovery:
    reagentc /disble
3. Check if the recovery partition exists:
    reagentc /info
4. Delete the partition if it exists:
    Get-Partition | Where-Object -FilterScript {$_.Type -eq 'Recovery'} | Remove-Partition -PassThru -Confirm:$false

    * if nothing is found, try:
    Get-Partition | Where-Object -FilterScript {$_.Type -eq 'Unknown'} | Remove-Partition -PassThru -Confirm:$false

#### 9 Install Windows Updates
Steps:
1. Open PowerShell as Administrator.
2. Run the following commands

    Add-WindowsCapability -Online -Name UpdateServices-Core
    Install-Module -Name PSWindowsUpdate -Force
    Import-Module PSWindowsUpdate
    Get-WindowsUpdate -Install -Verbose -AcceptAll -AutoReboot
3. Windows will install updates and restart. 

#### 10 Enable PowerShell Scripting

PowerShell is used for automation, but by default, Windows blocks scripts.

Steps:
1. Open PowerShell as Administrator
2. Run:
    Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser

#### 11 Enable Long File Paths
By default, Windows limits file paths to 260 characters. This step removes that restriction

Steps:
1. Open PowerShell as Administrator
2. Run:
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem' -Name "LongPathsEnabled" -Value 1
    New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force


#### 12 Disable Visual Effects (Performance Boost)
This removes unnecessary Windows animations for better performance.

Steps:
1. Open PowerShell as Administrator
2. Run:
    Set-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects' -Name "VisualFXSetting" -Value 2

#### 13 Set Environment Variables
This sets system-wide settings to optimize performance. 

Steps:
1. Open PowerShell as Administrator
2. Run:
    [System.Environment]::SetEnvironmentVariable('DOTNET_CLI_TELEMETRY_OPTOUT', 'true', [System.EnvironmentVariableTarget]::Machine)

#### 14 Remove Installation Media
Since the Windows and VirtIO drivers have been installed, remove the installation media. 

Steps:
1. In Proxmox, go to the Hardware tab for the Vm. 
2. Remove the two CD drives (Windows 11 ISo and VirtIO ISO).


#### 15 Install Security Software (If Required)
* if your school or workplace requires CrowdStrike Falcon, Install it now. 
* At BYU, IT policies require Falcon before allowing network access.

This completes the windows setup inside the VM! Now youre ready to install TeamCity and start configuring your DevOps pipeline.


#### What I need to do
* Woah, this is a lot. I basically need to just go through all of the documentation while I do this, I am not even gonna try and memorize all this.

#### What Connor had to say about the whole byu Falcon csr thing.
"Oh nah, it's nothing crazy. It's just worth noting that standard procedure at BYU is that any machines connected ot the local network via a wire connection need to have Falcon installed so they can get insights into what machines are on the network. 

Falcon is the antivirus BYU uses, but it also sends telemetry to the hub to keep tabs on everything in the event of a breach

But I don't use/pay for falcon on my personal machines and so I don't install it.

But when the build farm was housed at BYU, one of the first things I had to do was install falcon so they could whitelist the MAC address so the computer could actually connect to the internet. Then I could update the machine and install software on it. 


## Software Installation

#### Nuget
Install-PackageProvider -Name NuGet -Force

#### Chocolatey
* (Follow latest setup instructions at https://chocolatey.org/install)
* Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
* choco feature enable -n allowGlobalConfirmation

#### PowerShell 7
* choco install powershell-core
* Disable desktop on login, and just use powerShell (for performance reasons)
* Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name "Shell" -Value "pwsh.exe"
* If you REALLY want the desktop after this, just run explorer.exe in PowerShell

#### Media Feature Pack
* This is needed for certain online subsystems to work
* Get-WindowsCapability -online | Where-Object -Property name -like "*media*" | Add-WindowsCapability -Online

#### Perforce
* choco install p4

#### Direct X
*choco install directx

#### Visual Studio Build Tools
choco install visualstudio2022buildtools --package-parameters (($params = @(
    '--nocache'
    '--locale en-US'
    '--add Microsoft.Net.Component.4.8.1.TargetingPack'
    '--add Microsoft.Net.Component.4.8.1.SDK'
    '--add Microsoft.VisualStudio.Component.VC.14.38.17.8.x86.x64'
    '--add Microsoft.VisualStudio.Component.VC.14.38.17.8.ATL'
    '--add Microsoft.VisualStudio.Component.VC.Tools.x86.x64'
    '--add Microsoft.VisualStudio.Component.Windows11SDK.22621'
)) -join ' ')

#### Unreal Linux Toolchain
* choco install unreal-linux-toolchain
* Depending on the version of Unreal you are building, you may need to get an older version.
* Use the table here to decide: https://dev.epicgames.com/documentation/en-us/unreal-engine/linux-development-requirements-for-unreal-engine#versionhistory
* Then use choco to install the correct version, e.g. choco install unreal-linux-toolchain --version=22.16.0.6
* In theory you should be able to have multiple toolchains installed at once, but this is untested.

#### Windows Driver Kit
* This contains various tools like PDBCopy that are needed for building things like UGS binaries.
* choco install windowsdriverkit11

#### OctoBuild
* This emulates the XGE (Incredibuild) agent and acts as simple chaching system for build artifacts
* choco install octobuild
* [System.Environment]::SetEnvironmentVariable('OCTOBUILD_CACHE', 'C:\Cache\OctoBuild', [System.EnvironmentVariableTarget]::Machine)
* Configure the cache size to 512 GB (524288MB)
* [System.Environment]::SetEnvironmentVariable('OCTOBUILD_CACHE_LIMIT_MB', '524288', [System.EnvironmentVariableTarget]::Machine)
* This can help if you have multiple projects sharing the same engine version


#### SteamCMD
* Used to publish build to Steam
* choco install steamcmd
* Login to Steam:
    * steamcmd +login byudevops +quit
    * Enter the password, then enter the Steam Guard code sent via email
* The credentials should be cached from this point on. This is not an elegant solution, but it's what Valve recommends.


#### Sentry CLI
* Publishes debug symbols to Sentry, and notifies of new releasees
* choco install sentry-cli

#### TeamCity Agent
* choco install teamcityagent -params "agentDir=C:\\TeamCity agentWorkDir=C:\\Work serverUrl=<Server URL> agentName=<COMPUTER NAME>"
* The computer name should be a pretty name, so if the computer is "wardern-win" the agent name should be "Warden"
* The server URL should be the URL of the TeamCity server, e.g. "http://teamcity.cs.byu.edu:8111"
* Ideally, the server is setup behind a proxy and uses HTTPS, so the URL should be "https://teamcity.cs.byu.edu"
* After installing, you need to authorize the agent on the server. This can be done by going to the "Agents" tab on the server, finding the unauthorized agent, and clicking "Authorize..."

#### RESTART!
* shutdown /r /t 0

### ChatGPT analasys
