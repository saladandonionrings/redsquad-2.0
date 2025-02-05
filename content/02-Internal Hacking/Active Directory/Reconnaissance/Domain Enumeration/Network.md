# Enumerate alive machines
- Use Wireshark to identify some hosts and the network ranges

```bash
# with tcpdump
tcpdump -i <iface>

# with fping
fping -asgq <network-range>

# with zmap
sudo zmap -i $iface -P 2 --probe-module=icmp_echoscan -B 1M --max-targets=10000000 -o targets_rfc1918.txt $network_ips

# with arp-scan
arp-scan -d $networkrange

# with nxc - smb, ssh, rdp
nxc smb $networkrange

# responder
python3 Responder.py -I <iface> -rdwv
```

# Enumerate services
## DNS
```bash
# test for dns attacks
dnsenum $domain -f /usr/share/wordlists/SecLists/Discovery/DNS/namelist.txt --dnsserver $dns_server_ip > dnsenum.txt

# discover printers, web, shares, vpn, media
gobuster dns -d $domain -t 25 -w /opt/Seclist/Discovery/DNS/subdomain-top2000.txt
```

## Web
- If there is a lot of web servers on the network : https://github.com/sensepost/gowitness
```bash
# Search
gowitness file -f web-hosts --user-agent curl

# Report serve
gowitness report serve -a 0.0.0.0:7171
```