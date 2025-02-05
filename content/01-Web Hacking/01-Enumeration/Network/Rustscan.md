# RustScan Overview

>[!INFO]
>Rust Scan is a modern, fast, and efficient network scanner written in Rust. It aims to combine the speed of Rust with the power of Nmap to create a comprehensive scanning tool. Rust Scan is designed to be extremely fast and is capable of scanning an entire IP range or multiple IP addresses efficiently. 

## Key Features
- Ultra-fast port scanning.
- Integration with Nmap for detailed service and vulnerability scans.
- Support for multiple IP addresses and CIDR notation.
- Ability to scan selected ports or port ranges.
- Support for both TCP and UDP scans.

## Usage Instructions

### Basic Usage

#### Noisy Scan
For a comprehensive scan using Nmap's default scripts and service/version detection:
```bash
rustscan -a 0.0.0.0 -- -A -sC -sV -oN initial.log
```
- `-a 0.0.0.0`: Target IP address.
- `-A`: Enables OS detection, version detection, script scanning, and traceroute.
- `-sC`: Runs Nmap's default scripts.
- `-sV`: Version detection.
- `-oN initial.log`: Outputs the results to `initial.log`.

#### SYN "Stealth" Scan
To perform a SYN scan, which is less likely to be detected by firewalls:
```bash
sudo rustscan -a 0.0.0.0 -- -vv -oN Initial-SYN-Scan
```
- `-vv`: Increases verbosity.
- `-oN Initial-SYN-Scan`: Outputs the results to `Initial-SYN-Scan`.

#### Service Scan
To scan specific services:
```bash
sudo rustscan -a 0.0.0.0 -p 22,53,80,443 -- -sV -Pn -vv
```
- `-p 22,53,80,443`: Scans ports 22, 53, 80, and 443.
- `-sV`: Service version detection.
- `-Pn`: Disables host discovery, treating the targets as online.
- `-vv`: Increases verbosity.

### Advanced Usage

#### Multiple IP Scanning
To scan multiple IP addresses:
```bash
rustscan -a 0.0.0.0,1.1.1.1
```
- Scans both 0.0.0.0 and 1.1.1.1.

#### CIDR Support
To scan a range of IP addresses using CIDR notation:
```bash
rustscan -a 192.168.0.0/30
```
- Scans the IP range 192.168.0.0 to 192.168.0.3.

#### Selected Port Scanning
To scan specific ports on a target:
```bash
rustscan -a 0.0.0.0 -p 53,80,121,65535
```
- Scans ports 53, 80, 121, and 65535 on 0.0.0.0.

#### Port Range Scanning
To scan a range of ports:
```bash
rustscan -a 0.0.0.0 --range 1-1000
```
- Scans ports 1 through 1000 on 0.0.0.0.

#### UDP Scan
To perform a UDP scan:
```bash
rustscan -a 0.0.0.0 -sU -p ports
```
- `-sU`: Specifies a UDP scan.
- `-p ports`: Specifies the ports to scan.