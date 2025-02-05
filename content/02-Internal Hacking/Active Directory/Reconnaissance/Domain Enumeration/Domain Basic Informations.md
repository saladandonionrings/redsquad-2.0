# Windows

```bash
# Domain name
ipconfig /all

# Domain Controllers
nslookup <domain>
nslookup -type=SRV _ldap._tcp.dc.msdcs.<domain>

nltest /dclist:{domainname}

echo %logonserver%
```

# Linux

```bash
# Domain Name
cat /etc/resolv.conf

nxc smb <network-range>

# Domain Controllers
systemd-resolve --status | grep "DNS Servers"

nmcli dev show | grep DNS

# With nmap, usually the DCs have DNS, Kerberos and LDAP ports open
nmap -p53,88,389 $network_ip --open -v -oN DomainControllers
```
