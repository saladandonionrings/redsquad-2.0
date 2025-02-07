# SMB

>**SMB** stand for **S**erver **M**essage **B**lock, and it allows you to share your resources to other computers over the network. 
>There are 3 versions of SMB :
>* **SMBv1** version vulnerable to known exploits (Eternal Blue, Wanna Cry), now disabled by default in latest Windows version.
>* **SMBv2** reduced “chattiness” of SMB1. Guest access is disabled by default.
>* **SMBv3** guest access disabled, uses encryption. **Most secure**.
>
>TCP port **139** is SMB over NetBIOS, TCP port **445** is SMB over IP (latest version of SMB).
>List of SMB versions and corresponding Windows versions :
>* **SMB1** – Windows 2000, XP, and Windows 2003.
>* **SMB2** – Windows Vista SP1 and Windows 2008
>* **SMB2.1** – Windows 7 and Windows 2008 R2
>* **SMB3** – Windows 8 and Windows 2012.

## nmap
```bash
# Launch all scripts
nmap --script="smb-*" -p445 <target>
```

## Powerview
https://github.com/PowerShellMafia/PowerSploit/
```powershell
Import-Module .\PowerView.ps1

# Enumerate Shares
Find-DomainShare

# Listing contents 
ls \\<HOST>\<SHARE>\<DIRECTORY>\
```
## NXC

```bash
# No Account - Try with NULL or Guest
nxc smb <network> -u '' -p '' --shares
nxc smb <network> -u 'Guest' -p '' --shares

# User account
nxc smb <network> -u <user> -p <password> --shares
```

## smbclient-ng
https://github.com/p0dalirius/smbclient-ng

```bash
smbclient-ng -d <domain> -u <user> -p <password> --host <ip>
```

# FTP
>The **File Transfer Protocol (FTP)** serves as a standard protocol for file transfer across a computer network between a server and a client.
>It (usually) uses Port **21**.

## nmap
```bash
# Launch all scripts
nmap --script="ftp-*" <target>

# grab banner
nc <ip> 21
```
## NXC
```bash
# Test Anonymous binds and list files
nxc ftp <network> -u Anonymous -p ''

# Password spraying
nxc ftp <network> -u userslist -p passwordlist --no-bruteforce --continue-on-success

# List files
nxc ftp <network> -u username -p password --ls

# Download file
nxc ftp <network> -u username -p password --get <file>

# Upload file
nxc ftp <network> -u username -p password --put <local_file> <remote_file>
``` 
# NFS

```bash
# is there any nfs shares ?
showmount -e <ip>
# mount it
mount -t nfs -o rw,vers=2 <ip>:<remote_path> <local_path> -o nolock
```
