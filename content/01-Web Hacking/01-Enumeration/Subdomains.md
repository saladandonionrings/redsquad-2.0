# Online
- https://dnsdumpster.com/
- https://subdomainfinder.c99.nl/
# Sublist3r 
## Overview

>[!What is Sublist3r ?]
>Sublist3r is a Python-based tool designed to enumerate subdomains of websites using OSINT (Open Source Intelligence). It is particularly useful for penetration testers and bug hunters to collect and gather subdomains for the domains they are targeting. Sublist3r leverages multiple search engines and online services to find subdomains, including Google, Yahoo, Bing, Baidu, Ask, Netcraft, VirusTotal, ThreatCrowd, DNSdumpster, and ReverseDNS. Additionally, Sublist3r integrates with Subbrute to enhance subdomain discovery through brute-force techniques using an improved wordlist.

## Key Features
- Enumerates subdomains using multiple search engines.
- Utilizes online services for comprehensive subdomain discovery.
- Integrates brute-force techniques for increased subdomain detection.
- Easy to install and use.

## Installation
```bash
git clone https://github.com/aboul3la/Sublist3r.git
cd Sublist3r
pip install -r requirements.txt
python3 sublist3r.py -h
```

## Usage Examples

### Basic Subdomain Enumeration
To enumerate subdomains for a specific domain:
```bash
python3 sublist3r.py -d example.com
```

### Using a Specific Search Engine
To use a specific search engine, such as Google, for subdomain enumeration:
```bash
python3 sublist3r.py -d example.com -e google
```

### Brute-Force Subdomain Enumeration
To perform brute-force subdomain enumeration using Subbrute:
```bash
python3 sublist3r.py -d example.com -b
```

## Other DNS Recon Tools

### DNSRecon
DNSRecon is another tool for brute-forcing subdomains:
```bash
dnsrecon -d example.com -t brt -D /usr/share/wordlists/dnsmap.txt
```

### DNSEnum
DNSEnum is used for comprehensive DNS enumeration:
```bash
dnsenum example.com
```

### Fierce
Fierce is a DNS reconnaissance tool with a brute-force option:
```bash
fierce -dns example.com -wordlist dictionary.txt
```

## Wfuzz Overview

Wfuzz is a versatile tool used for brute-forcing web applications, including subdomain discovery. It allows customization of headers, such as the Host header, and filtering of HTTP response codes to identify valid subdomains.

### Basic Usage Examples

#### Subdomain Brute-Forcing
To brute-force subdomains using a wordlist:
```bash
wfuzz -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.example.com" --hc 403,400 -t 80 example.com
```

#### Brute-Forcing with Custom Headers
To brute-force with a custom Host header:
```bash
wfuzz -c -w wordlist.txt -u http://example.com -H "Host: FUZZ.shoppy.htb" --hc 301
```

### Additional Tips

#### Modify /etc/hosts
To ensure subdomain resolution, you can add entries to the `/etc/hosts` file:
```bash
echo "10.10.11.180 mattermost.shoppy.htb" | sudo tee -a /etc/hosts
```
