# OpenSSH-PowerShell-remoting-Windows-machines
- OpenSSH+PowerShell remoting Windows machines

## Step by step setup **SSH** + **PowerShell remoting** to Windows machines.
---
### 🔹 1. Install OpenSSH Server on the remote Windows machine

On **Windows 10/11** or **Server 2019+**:

```powershell
# Run in an elevated PowerShell prompt
# List ...
Get-WindowsCapability -Online | Where({ $_.Name -like 'openssh*' }) | ft
# shows, for example
Name                           State
----                           -----
OpenSSH.Client~~~~0.0.1.0  Installed
OpenSSH.Server~~~~0.0.1.0 NotPresent

# add new
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
# should result in
Name         : OpenSSH.Server~~~~0.0.1.0
State        : Installed
DisplayName  : OpenSSH-Server
Description  : OpenSSH-basierter Secure Shell (SSH)-Server für die sichere Schlüsselverwaltung und für den Zugriff auf
               Remotecomputer
DownloadSize : 1290075
InstallSize  : 4947215
```

**Hint:**
- If there is an error, like
Access denied
    + CategoryInfo          : NotSpecified: (:) [Add-WindowsCapability], COMException
    + FullyQualifiedErrorId : Microsoft.Dism.Commands.AddWindowsCapabilityCommand

Try again to remote connect with PSExec64.exe using elevation -h and system credentials -s, like ...
```powershell
PsExec64.exe \\Remote-Computer-Name -h -s powershell -noexit -command "whoami"

# a check should now reveal elevated System-Account rights ...
PS C:\WINDOWS\system32>whoami
nt-autorität\system

# adding OpenSSH.Server should now work
PS C:\WINDOWS\system32> Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
...
Path          :
Online        : True
RestartNeeded : False

PS C:\WINDOWS\system32> Get-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Name         : OpenSSH.Server~~~~0.0.1.0
State        : Installed
DisplayName  : OpenSSH-Server
Description  : OpenSSH-basierter Secure Shell (SSH)-Server für die sichere Schlüsselverwaltung und für den Zugriff auf
               Remotecomputer
DownloadSize : 1290075
InstallSize  : 4947215
```

Then start and enable the service:
```powershell
Start-Service sshd

Get-Service sshd
Status   Name               DisplayName
------   ----               -----------
Running  sshd               OpenSSH SSH Server

Set-Service -Name sshd -StartupType 'Automatic'
```

(Optionally allow the firewall rule: `New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22`)

---

## 🔹 2. Test SSH login

From your client (Linux, macOS, or Windows with OpenSSH installed):

```bash
ssh user@remote-windows-host
```

If successful, you’ll land in a **PowerShell session** on the remote machine.

---

## 🔹 3. Enable PowerShell Remoting over SSH

By default, **PowerShell Remoting** uses **WinRM** (not SSH). But you can configure it to run over SSH:

1. Create a PowerShell endpoint that uses SSH transport:

```powershell
# On remote Windows host
New-PSSessionConfigurationFile -Path ./ssh_endpoint.pssc -RunAsVirtualAccount
Register-PSSessionConfiguration -Name SSH -Path ./ssh_endpoint.pssc
```

2. Now you can connect from your client:

```powershell
# From client machine
Enter-PSSession -HostName remote-windows-host -UserName user -SSHTransport
```

Or run one-off commands:

```powershell
Invoke-Command -HostName remote-windows-host -UserName user -SSHTransport -ScriptBlock { Get-Process }
```

---

## 🔹 4. Authentication

* Default: **password-based SSH login**
* Recommended: **key-based authentication**

  * Generate SSH keys (`ssh-keygen`) on client.
  * Place your public key into `C:\Users\<username>\.ssh\authorized_keys` on the remote Windows machine.

---

## 🔹 5. Verify it works

After connecting, test with a simple cmdlet:

```powershell
Enter-PSSession -HostName remote-windows-host -UserName user -SSHTransport
# Inside session:
Get-ComputerInfo
```

---

✅ At this point, you can manage the Windows machine remotely with **SSH + PowerShell remoting**, all text-based and secure.

---

Would you like me to also give you a **ready-to-use setup script** for the remote machine (so you don’t have to type each command manually), or do you prefer manual step-by-step control?

Would you like me to also give you a ready-to-use setup script for the remote machine (so you don’t have to type each command manually), or do you prefer manual step-by-step control?
