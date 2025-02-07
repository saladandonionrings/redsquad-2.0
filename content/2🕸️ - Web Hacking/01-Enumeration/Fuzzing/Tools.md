# Feroxbuster 
## Overview

>[!INFO] 
>Feroxbuster is a simple, fast, and recursive content discovery tool written in Rust. It is designed to help security professionals and enthusiasts in discovering hidden content on web servers by recursively crawling URLs. The tool supports both HTTP and HTTPS protocols and allows users to specify various options such as the number of threads, target URL, wordlists, file extensions, and output files.

## Features
- Written in Rust for performance and reliability.
- Supports recursive content discovery.
- Customizable with various options and wordlists.
- Handles both HTTP and HTTPS protocols.
- Ability to specify file extensions and output results to a file.

## Usage Instructions
### HTTP
#### Using SecLists
```bash
feroxbuster -t 10 -u http://$target -w /usr/share/seclists/Discovery/Web-Content/common.txt -o feroxbuster
```
- `-t 10`: Specifies the number of threads.
- `-u http://$target`: Specifies the target URL.
- `-w /usr/share/seclists/Discovery/Web-Content/common.txt`: Specifies the wordlist to use.
- `-o feroxbuster`: Specifies the output file.

#### Using DirBuster
```bash
feroxbuster -t 10 -u http://$target -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o feroxbuster
```
- Uses the DirBuster wordlist for content discovery.

#### Specifying File Extensions
```bash
feroxbuster -t 10 -u http://$target -w /usr/share/seclists/Discovery/Web-Content/common.txt -x py,html,txt -o feroxbuster
```
- `-x py,html,txt`: Specifies the file extensions to search for.

### HTTPS

#### Using SecLists with HTTPS
```bash
feroxbuster -t 10 -u https://$target -k -w /usr/share/seclists/Discovery/Web-Content/common.txt -o feroxbuster
```
- `-k`: Ignores SSL certificate verification.
- Other options remain the same as the HTTP example.

#### Specifying File Extensions with HTTPS
```bash
feroxbuster -t 10 -u https://$target -k -w /usr/share/seclists/Discovery/Web-Content/common.txt -x py,html,txt -o feroxbuster
```
- Same as the previous example but includes file extension specification.

# Gobuster
## Overview

>[!INFO]
>Gobuster is a robust and versatile tool designed for brute-forcing URIs (directories and files), DNS subdomains, and virtual host names. It is widely used by security professionals for web application testing and reconnaissance. For this guide, the focus will be on using Gobuster to brute-force directories.

## Key Features
- Brute-force directories and files.
- Enumerate DNS subdomains.
- Fuzzing capabilities.
- AWS S3 bucket enumeration.
- Virtual host enumeration.

## Modules

Gobuster includes several modules, each targeting different types of enumeration:

```bash
dir    # Uses directory/file enumeration mode
dns    # Uses DNS subdomain enumeration mode
fuzz   # Uses fuzzing mode
help   # Help about any command
s3     # Uses AWS bucket enumeration mode
version # Shows the current version
vhost  # Uses virtual host enumeration mode
```

## Flags

Gobuster offers a variety of global flags for customization and output control:

```bash
--delay <duration>       # Time each thread waits between requests (e.g., 1500ms)
-h, --help               # Help for gobuster
    --no-error           # Don't display errors
-z, --no-progress        # Don't display progress
-o, --output <string>    # Output file to write results
-p, --pattern <string>   # File containing replacement patterns
-q, --quiet              # Don't print the banner and other noise
-t, --threads <int>      # Number of concurrent threads (default 10)
-v, --verbose            # Verbose output (errors)
-w, --wordlist <string>  # Path to the wordlist
```

### Examples

#### Directory Enumeration with Wordlist Selection
```bash
gobuster dir -w `fzf-wordlists` -u http://0.0.0.0
```
- Prompts for wordlist selection using `fzf-wordlists`.

#### Directory Enumeration with Specified File Extensions
```bash
gobuster dir -u http://0.0.0.0 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,js,json,php,py
```
- `-x txt,html,js,json,php,py`: Specifies the file extensions to search for.

#### Directory Enumeration Ignoring Specific Status Codes
```bash
gobuster dir -u http://0.0.0.0 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -d 403 404
```
- `-d 403 404`: Ignores HTTP status codes 403 and 404.

#### DNS Subdomain Enumeration
```bash
gobuster dns -d http://0.0.0.0 -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```
- Uses the DNS module with a specified wordlist for subdomain enumeration.

#### Directory Enumeration with Proxy
```bash
gobuster dir -u http://0.0.0.0 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,js,json,php,py --proxy http://127.0.0.1:8081
```
- Uses a proxy for the requests.

# Wfuzz
## Overview

>[!INFO]
>Wfuzz is a powerful web application brute-forcing tool that allows for comprehensive web directory and file fuzzing. It is particularly useful for discovering hidden directories, files, and parameter brute-forcing in web applications. Wfuzz provides a flexible way to automate and customize web fuzzing tasks.

## Key Features
- Directory and file brute-forcing.
- Support for custom wordlists.
- Ability to filter HTTP response codes.
- Capable of handling multiple payloads and request customization.
- Open-source and extensible.

## Usage Instructions

### Basic Directory Fuzzing
To search for directories and ignore 404 responses:
```bash
wfuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hc 404 http://$target/FUZZ
```
- `-w`: Specifies the wordlist to use.
- `--hc 404`: Hides responses with the 404 status code.
- `http://$target/FUZZ`: Specifies the target URL with the fuzzing placeholder.

### Searching for Specific File Types
To search for PHP files:
```bash
wfuzz -w wordlist/general/common.txt http://$target/FUZZ.php
```
- `http://$target/FUZZ.php`: Specifies the target URL pattern, searching for PHP files.

### Brute-Forcing with Multiple Payloads
To use two wordlists for username and password and ignore 302 responses:
```bash
wfuzz -z file,/usr/share/wordlists/rockyou.txt -d "uname=FUZZ&pass=FUZZ" --hc 302 http://$target/userinfo.php
```
- `-z file,/usr/share/wordlists/rockyou.txt`: Specifies the wordlist for payloads.
- `-d "uname=FUZZ&pass=FUZZ"`: Specifies the data to be sent in the request with fuzzing placeholders.
- `--hc 302`: Hides responses with the 302 status code.

# FFUF 
## Overview

>[!INFO]
>FFUF (Fuzz Faster U Fool) is another web fuzzing tool designed for speed and efficiency. It supports various modes of operation, including URL fuzzing, directory brute-forcing, and parameter fuzzing. FFUF is known for its performance and flexibility.

## Key Features
- High-speed web fuzzing.
- Support for custom wordlists.
- Filtering and matching HTTP response codes and sizes.
- Versatile use cases, including URL, directory, and parameter fuzzing.
- Open-source and easy to install.

## Installation Instructions
### Step-by-Step Guide
1. Update package list and install Go:
```bash
apt update && apt install go
```
1. Clone the FFUF repository:
```bash
git clone https://github.com/ffuf/ffuf
```
1. Navigate to the FFUF directory:
```bash
cd ffuf
```
1. Get the dependencies and build the project:
```bash
go get
go build
```

## Usage Instructions
### Basic Directory Fuzzing
To fuzz a target URL and exclude specific response codes:
```bash
ffuf -u https://$target/FUZZ -w raft-large-directories.txt -fc 401,403,404 -fs 0
```
- `-u https://$target/FUZZ`: Specifies the target URL with the fuzzing placeholder.
- `-w raft-large-directories.txt`: Specifies the wordlist to use.
- `-fc 401,403,404`: Filters out responses with status codes 401, 403, and 404.
- `-fs 0`: Filters out responses with a size of 0 bytes.