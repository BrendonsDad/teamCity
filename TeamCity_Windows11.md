# TeamCity Agent Setup (Windows 11)

# VM Setup

### Resources
- Download Windows 11 iso: https://www.microsoft.com/en-gb/software-download/windows11
- Download VirtIO drivers: https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso
- Upload images to Proxmox

### VM Creation
- Create a new VM called on the desired node with the following options:
  - General/Name: `windows`
  - OS/Iso Image: (Windows 11 iso uploaded earlier)
  - OS/Type: `Windows`
  - OS/Version: `11/2022/2025`
  - OS/Add additional drive for VirtIO drivers: `true`
  - OS/VirtIO ISO: (VirtIO drivers iso uploaded earlier)
  - System/Qemu Agent: `true`
  - System/EFI Storage: `local-lvm`
  - System/TPM Storage: `local-lvm`
  - Disks/Storage: `local-lvm`
  - Disk/Disk size: (The largest you can make it, typically 90% of the total storage of the host)
  - CPU/Cores: (Same as the host machine)
  - CPU/Type: `host`
  - Memory/Memory: (Same as the host machine, minus 1GB to leave room for the hypervisor)

One the machine is created, go to the Options page for the VM and set "Start at boot" to true.

# Windows Install

If you are using Windows Desktop rather than Windows Server, there are some extra steps to bypass all the OOB (out of box) experience stuff. YOU DO NOT WANT A MICROSOFT ACCOUNT ON THIS MACHINE.

Make sure to install "Windows 11 Pro" or "Windows 11 Enterprise", NOT "Windows 11 Home".

When it looks for a disk to install windows on, it will fail, because the VirtIO drivers are not installed. Click "Load Driver" and navigate to the VirtIO CD drive and install the drivers.
It should be something like "D:\amd64\w11". Once you select that folder the driver "Red Hat VirtIO SCSI pass-through controller" should appear. Install that driver and then click "Install".

# Windows Setup

It may fail to connect to a network because it needs the network driver. Similar to the disk driver, click "Install driver" and navigate to the VirtIO CD drive and install the network driver. It will most likely make you reboot after it finds the driver.

When it comes time to setup the user, we only plan on using the Administrator account, so the account we create using the OOB experience will be deleted. So it doesn't really matter how you create it.
If it forces you to create a user linked to a Microsoft account, you will need to bypass this.
1. Press "Shift+F10" to open a command prompt.
2. Run: `OOBE\BYPASSNRO`
3. Screw Microsoft.

The placeholder user we create will be called "Steve" for historical reasons. It will be deleted later, and all the scripts will assume this user to be Steve.
Just fill out all the security questions with "steve" to get it over with.
Turn off ALL telemetry and data collection. You don't want Microsoft knowing what you're doing.

# Post-Install

Enable the Administrator account:
- Open a command prompt as Administrator
- Run: `net user Administrator /active:yes`
- Run: `net user Administrator *` (Set a password)
- Run: `net user Steve /delete`
- Reboot and login with the Administrator account

Set the host name using PowerShell:
- `Rename-Computer -NewName "<HOST>-win" -Restart`
- (If the host name is "warden", the computer name should be "warden-win")
- (so run `Rename-Computer -NewName "warden-win" -Restart`)

Set timezone to mountain time (if not already set)
`tzutil /s "Mountain Standard Time"`

### Install remaining VirtIO drivers
- `msiexec /i D:\virtio-win-gt-x64.msi /passive`
- `D:\virtio-win-guest-tools.exe /passive`
- `Restart` (you can also unmount the VirtIO CD drive in Proxmox)

### Enable RDP
- `Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0`
- (Alternatively, you can use the GUI by searching for "Remote Desktop" in the settings)
- Restart

Going forward, you can use RDP to connect to the machine. This makes copying and pasting much easier.
If the session disconnects due to "encryption failure" check out this article (https://superuser.com/questions/1747488)

**FROM THIS POINT ON USER POWERSHELL INSTEAD OF CMD!!!!!**

### Configure automatic login:
- `reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoAdminLogon /t REG_SZ /d 1 /f`
- `reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName /t REG_SZ /d Administrator /f`
- `reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword /t REG_SZ /d <YourPasswordHere> /f`

### De-bloat Windows (optional, but recommended)
- Open PowerShell
- `irm "https://christitus.com/win" | iex`
- Configure the "Tweaks" to your liking

### Remove WinRE
- `reagentc /disable`
(Check status with `reagentc /info`)
- `Get-Partition | Where-Object -FilterScript {$_.Type -eq 'Recovery'} | Remove-Partition -PassThru -Confirm:$false`
(OPTIONALLY: If that didn't find anything, try removing unknown partitions)
- `Get-Partition | Where-Object -FilterScript {$_.Type -eq 'Unknown'} | Remove-Partition -PassThru -Confirm:$false`

### Install Updates
- `Add-WindowsCapability -Online -Name UpdateServices-Core`
- `Install-Module -Name PSWindowsUpdate -Force`
- `Import-Module PSWindowsUpdate`
- `Get-WindowsUpdate -Install -Verbose -AcceptAll -AutoReboot`
- (NOTE: Update KB5034439, if applicable, may fail to install, this is fine.)
- This process can take a while.

### Allow PowerShell Scripting
- `Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser`

### Enable Long Paths (Important!)
- `Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem' -Name "LongPathsEnabled" -Value 1`
- `New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force`

### Disable Visual Effects
- `Set-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects' -Name "VisualFXSetting" -Value 2`

### Misc Environment Variables
- `[System.Environment]::SetEnvironmentVariable('DOTNET_CLI_TELEMETRY_OPTOUT', 'true', [System.EnvironmentVariableTarget]::Machine)`

### Remove Install Media
- In Proxmox, go to the Hardware tab for the VM and remove the two CD drives. (Window11 and VirtIO)

### Setup CrowdStrike
- We don't use this at Two Giants, but if your IT policy requires it, do it now.
- At BYU, the CSRs wouldn't whitelist the MAC address of the VM until Falcon was installed.

# Software Installation

### Nuget
`Install-PackageProvider -Name NuGet -Force`

### Chocolatey
- (Follow latest setup instructions at https://chocolatey.org/install)
- `Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))`
- `choco feature enable -n allowGlobalConfirmation`

### PowerShell 7
- `choco install powershell-core`
- Disable desktop on login, and just use PowerShell (for performance reasons)
- `Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name "Shell" -Value "pwsh.exe"`
- If you _REALLY_ want the desktop after this, just run `explorer.exe` in PowerShell

### Media Feature Pack
- This is needed for certain online subsystems to work
- `Get-WindowsCapability -online | Where-Object -Property name -like "*media*" | Add-WindowsCapability -Online`

### Perforce
- `choco install p4`

### DirectX
- `choco install directx`

### Visual Studio Build Tools
```powershell
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
```

These components are based on these requirements: https://github.com/EpicGames/UnrealEngine/blob/ue5-main/Engine/Config/Windows/Windows_SDK.json

### Unreal Linux Toolchain
- `choco install unreal-linux-toolchain`
- Depending on the version of Unreal you are building, you may need to get an older version.
- Use the table here to decide: https://dev.epicgames.com/documentation/en-us/unreal-engine/linux-development-requirements-for-unreal-engine#versionhistory
- Then use choco to install the correct version, e.g. `choco install unreal-linux-toolchain --version=22.16.0.6`
- In theory you should be able to have multiple toolchains installed at once, but this is untested.

### Windows Driver Kit
- This contains various tools like PDBCopy that are needed for building things like UGS binaries.
- `choco install windowsdriverkit11`

### OctoBuild
- This emulates the XGE (Incredibuild) agent and acts as a simple caching system for build artifacts.
- `choco install octobuild`
- `[System.Environment]::SetEnvironmentVariable('OCTOBUILD_CACHE', 'C:\Cache\OctoBuild', [System.EnvironmentVariableTarget]::Machine)`
- Configure the cache size to 512GB (524288MB)
- `[System.Environment]::SetEnvironmentVariable('OCTOBUILD_CACHE_LIMIT_MB', '524288', [System.EnvironmentVariableTarget]::Machine)`
- This can help if you have multiple projects sharing the same engine version.

### SteamCMD
- Used to publish builds to Steam
- `choco install steamcmd`
- Login to Steam:
  - `steamcmd +login byudevops +quit`
  - Enter the password, then enter the Steam Guard code sent via email
- The credentials _should_ be cached from this point on. This is not an elegant solution, but it's what Valve recommends.

### Sentry CLI
- Publishes debug symbols to Sentry, and notifies of new releases.
- `choco install sentry-cli`

### TeamCity Agent
- `choco install teamcityagent -params "agentDir=C:\\TeamCity agentWorkDir=C:\\Work serverUrl=<SERVER URL> agentName=<COMPUTER NAME>"`
- The computer name should be a pretty name, so if the computer is "warden-win", the agent name should be "Warden"
- The server URL should be the URL of the TeamCity server, e.g. "http://teamcity.cs.byu.edu:8111"
- Ideally, the server is setup behind a proxy and uses HTTPS, so the URL should be "https://teamcity.cs.byu.edu"
- After installing, you need to authorize the agent on the server. This can be done by going to the "Agents" tab on the server, finding the unauthorized agent, and clicking "Authorize..."

**RESTART!**
- `shutdown /r /t 0`

# Post-Software Installation

### Windows Defender Config
- `Set-MpPreference -ExclusionPath "C:\TeamCity" -CloudBlockLevel 0 -SubmitSamplesConsent 2`
- `Set-MpPreference -ExclusionPath "C:\Work" -CloudBlockLevel 0 -SubmitSamplesConsent 2`
- Or just disable it entirely if you have a different AV solution:
- `Remove-WindowsFeature Windows-Defender` (Server Core)
- `Remove-WindowsFeature Windows-Defender-GUI` (Server DE)
- `Get-Service WinDefend | Stop-Service -PassThru | Set-Service -StartupType Disabled` (Desktop)
  - If Windows blocks that command, you can just disable components of Windows Defender:
  - `Set-MpPreference -DisableRealtimeMonitoring $true`
  - `Set-MpPreference -DisableBehaviorMonitoring $true`
  - `Set-MpPreference -DisableScriptScanning $true`
  - `New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender" -Name DisableAntiSpyware -Value 1 -PropertyType DWORD -Force`

### Create SysPrep Image
- Not needed if you just plan on doing this manually.

# Updating

You should try to update the image every month or so.

Choco makes it easy to update software it installed:
- `choco upgrade all -y`

To update Windows itself, just repeat the steps in the "Install Updates" section.
