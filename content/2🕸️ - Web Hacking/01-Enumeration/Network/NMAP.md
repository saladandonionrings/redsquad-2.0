# Nmap Overview

>[!INFO]
>Nmap (Network Mapper) is a powerful open-source tool used for network discovery and security auditing. It is widely used for network inventory, managing service upgrade schedules, and monitoring host or service uptime. Nmap can be used to scan a wide range of devices and provides various functionalities such as port scanning, version detection, OS detection, and more.

## Key Features
- Port scanning to identify open ports on a network.
- Service version detection.
- OS detection and fingerprinting.
- Scripting engine for advanced service detection and vulnerability scanning.
- Flexible output options for report generation.

## Usage Instructions

### Default Nmap Script Scan
Perform a comprehensive scan using default scripts and service version detection:
```bash
sudo nmap -sV -sC -p- 0.0.0.0
```
- `-sV`: Attempts to determine the version of the services running.
- `-sC`: Runs default scripts.
- `-p-`: Scans all ports.

For a faster scan with a minimum rate of 1000 packets per second:
```bash
nmap -T4 -sC -sV -p- --min-rate=1000 0.0.0.0
```
- `-T4`: Sets the timing template to 4 (higher is faster).
- `--min-rate=1000`: Sets the minimum packet rate to 1000 packets per second.

### Banner Grabbing
To grab service banners on a specific port:
```bash
nmap -sV --script=banner -p21 0.0.0.0/24
```
- `--script=banner`: Uses the banner grabbing script.

Using netcat for banner grabbing:
```bash
nc -nv 0.0.0.0
netcat 0.0.0.0 port
```

### TCP, FTP, and SMB Scanning
#### TCP Scan
```bash
nmap -Pn -sT -sC -sV -p0-65535 0.0.0.0
```
- `-Pn`: Disables host discovery.
- `-sT`: Performs a TCP connect scan.

#### FTP Scan
```bash
nmap -sC -sV -p21 0.0.0.0
```
- Scans for FTP services on port 21.

#### SMB Scan
```bash
nmap --script smb-os-discovery.nse -p445 0.0.0.0
```
- Uses the SMB OS discovery script.

## Common Flags
```bash
-sV          # Service version detection
-p <x>       # Port scan for port x or scan all ports
-p-          # Scan all ports
-Pn          # Disable host discovery
-A           # OS and version detection, script scanning
-sC          # Default script scan
-oN <file>   # Normal output
-oA <file>   # Output in all formats
-v           # Verbose mode
-sU          # UDP port scan
-sS          # TCP SYN scan
-T1-4        # Timing template (higher is faster)
-D <IP>      # Decoy scan using specified IP
```

## Firewall Evasion
```bash
-Pn
-f                  # Fragment packets
--mtu <number>      # Set maximum transmission unit size
--scan-delay <time> # Delay between packets
--badsum            # Send packets with bad checksums
```

Example: Scanning from a spoofed IP:
```bash
nmap 192.168.1.1 -D 192.168.1.2
```

Example: Scanning Facebook from Microsoft:
```bash
nmap -S www.microsoft.com www.facebook.com
```

Example: Using a specific source port:
```bash
nmap 192.168.1.1 -g 53
```

## Scripting
```bash
-sC                         # Default NSE scripts
--script default            # Default scripts
--script=banner             # Single script
--script=http*              # Wildcard script
--script=http,banner        # Multiple scripts
--script "not intrusive"    # Exclude intrusive scripts
--script-args <args>        # Script arguments
```

## Example Commands
### HTTP Site Map Generator
```bash
nmap -Pn --script=http-sitemap-generator scanme.nmap.org
```

### Fast Search for Random Web Servers
```bash
nmap -n -Pn -p 80 --open -sV -vvv --script banner,http-title -iR 1000
```

### DNS Brute Force
```bash
nmap -Pn --script=dns-brute domain.com
```

### Safe SMB Scripts
```bash
nmap -n -Pn -vv -O -sV --script smb-enum*,smb-ls,smb-mbenum,smb-os-discovery,smb-s*,smb-vuln*,smbv2* -vv 192.168.1.1
```

### Whois Query
```bash
nmap --script whois* domain.com
```

### Detect Cross-Site Scripting Vulnerabilities
```bash
nmap -p80 --script http-unsafe-output-escaping scanme.nmap.org
```

### Check for SQL Injections
```bash
nmap -p80 --script http-sql-injection scanme.nmap.org
```
