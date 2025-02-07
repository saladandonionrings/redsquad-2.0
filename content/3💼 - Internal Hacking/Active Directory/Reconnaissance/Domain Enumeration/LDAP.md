# Overview

> **LDAP** (Lightweight Directory Access Protocol) is a software protocol for enabling anyone to **locate data** about **organizations**, **individuals** and other resources such as files and devices in a network - whether on the public Internet or on a corporate Intranet.
> LDAP is a "**lightweight**" (smaller amount of code) version of Directory Access Protocol (DAP), which is part of X.500, a standard for directory services in a network.

## Ports

* **389** : LDAP (regular)
* **636** : LDAPs (LDAP over TLS/SSL)
* **3268** : msft-gc, Microsoft Global Catalog (LDAP service which contains data from Active Directory forests)
* **3269** : msft-gc-ssl, Microsoft Global Catalog over SSL (similar to port 3268, LDAP over SSL)

## Enumerate

```bash
# nmap
nmap -n -sV --script "ldap* and not brute" -p 389 $dcip
```

### Anonymous bind
>LDAP Anonymous binds allow unauthenticated attackers to retrieve information from the domain (users, groups, computers lists, etc.). This is a **legacy configuration**, and as of Windows Server 2003, only authenticated users are permitted to initiate LDAP requests.

```bash
# Get everything
ldapsearch -x -H ldap://<dcip> -b "dc=domain,dc=local" "objectclass=*"

# List users
ldapsearch -x -H ldap://<dcip> -b "dc=domain,dc=local" "objectclass=User"
# sAMAccountName
ldapsearch -x -H ldap://<dcip> -b "dc=htb,dc=local" "objectclass=User" | grep "sAMAccountName:" | awk '{print $2}'
```

## ldeep

> In-depth LDAP enumeration utility. Complete and updated.

https://github.com/franc-pentest/ldeep

```bash
# usage
ldeep ldap -s ldap://<ldapserverip> -u <user> -p <password> -d ';' all ldeep-output
```

## ldapdomaindump

> ldapdomaindump is a tool which aims to solve this problem, by collecting and parsing information available via LDAP and outputting it in a readable HTML format, as well as machine-readable JSON and CSV/TSV/greppable files. \
> **Alternative of ldapsearch**

https://github.com/dirkjanm/ldapdomaindump

```bash
ldapdomaindump -u <domain>\\<user> -p <password> -d ';' ldap://<ldapserverip>
```
