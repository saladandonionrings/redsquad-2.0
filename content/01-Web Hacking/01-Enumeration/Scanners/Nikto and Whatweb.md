# Nikto Overview

>[!INFO]
>Nikto is a web server scanner designed to identify vulnerabilities and potential issues on a given web server. It scans for various types of vulnerabilities including outdated server software, server misconfigurations, default files, and potentially dangerous files or programs. 

## Key Features
- Comprehensive scan for vulnerabilities.
- Supports SSL-enabled scans.
- Extensible via plugins.
- Open-source and actively maintained.

## Usage Instructions

### Basic Scan
To perform a basic scan on a domain:
```bash
nikto -h http://0.0.0.0
```
- `-h http://0.0.0.0`: Specifies the target domain to scan.

### SSL-Enabled Scan
To scan a domain with SSL enabled:
```bash
nikto -h https://0.0.0.0 -ssl
```
- `-ssl`: Indicates that the target domain uses SSL.

## WhatWeb Overview

WhatWeb is a tool for identifying web technologies used on websites. It recognizes a wide range of web technologies, including content management systems (CMS), web servers, JavaScript libraries, and more. WhatWeb has over 900 plugins to identify various technologies and provides detailed information about the target web server.

### Key Features
- Recognizes a wide range of web technologies.
- Identifies version numbers, email addresses, account IDs, web framework modules, and SQL errors.
- Over 900 plugins for comprehensive identification.
- Supports different levels of scan aggression.

## Usage Instructions

### Basic Scan
To perform a basic scan on a single domain:
```bash
whatweb 0.0.0.0
```

### Scan with No Errors
To scan a range of IP addresses and suppress errors:
```bash
whatweb --no-errors 10.10.10.0/24
```
- `--no-errors`: Suppresses error messages.

### Aggression Levels
To perform scans with different levels of aggression:
```bash
whatweb --aggression=Stealthy/Aggressive/Heavy --verbose
```
- `--aggression`: Sets the level of scan aggression (Stealthy, Aggressive, or Heavy).
- `--verbose`: Enables verbose output.

### Examples

#### Scan a Single Domain
```bash
whatweb example.com
```

#### Scan Multiple Domains Verbosely
```bash
whatweb -v reddit.com slashdot.org
```
- `-v`: Enables verbose plugin descriptions.

#### Aggressive Scan
```bash
whatweb -a 3 www.wired.com
```
- `-a 3`: Performs an aggressive scan to detect the exact version of technologies.

#### Scan Local Network Suppressing Errors
```bash
whatweb --no-errors 192.168.0.0/24
```

#### Scan Local Network for HTTPS Websites
```bash
whatweb --no-errors --url-prefix https:// 192.168.0.0/24
```
- `--url-prefix https://`: Scans for HTTPS websites.
