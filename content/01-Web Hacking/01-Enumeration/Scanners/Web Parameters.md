# Arjun

>[!INFO]
>Arjun is a tool designed to find hidden parameters on web pages. It is particularly useful for security testers who need to discover potential hidden inputs that could be manipulated for security testing. Arjun automates the process of sending requests to endpoints and checking for hidden parameters.

## Key Features
- Detects hidden parameters in web applications.
- Supports both GET and POST methods.
- Multi-threading for faster scans.
- Adjustable delay between requests.
- Easy to install and use.

## Installation Instructions

### Using pip
1. Install Arjun using pip:
```bash
# with pip
pip3 install arjun

# from source
git clone https://github.com/s0md3v/Arjun.git
python3 setup.py install
```

## Usage Instructions

### Basic Usage
To find hidden parameters on a specific endpoint using the GET method:
```bash
python3 arjun.py -u https://api.example.com/endpoint --get
```
- `-u https://api.example.com/endpoint`: Specifies the target URL.
- `--get`: Uses the GET method for requests.

### Multi-Threading
To enable multi-threading for faster scans:
```bash
python3 arjun.py -u https://api.example.com/endpoint --get -t 22
```
- `-t 22`: Specifies the number of threads to use.

### Delay Between Requests
To set a delay between requests to avoid rate limiting or detection:
```bash
python3 arjun.py -u https://api.example.com/endpoint --get -d 2
```
- `-d 2`: Sets a delay of 2 seconds between requests.

## Burp Suite Param Miner

>Burp Suite's Param Miner extension is a powerful tool for discovering hidden parameters and testing for HTTP parameter pollution (HPP). It is available as a plugin for the Burp Suite web vulnerability scanner.

### Key Features
- Detects hidden parameters in HTTP requests.
- Identifies potential parameter pollution vulnerabilities.
- Integrates with Burp Suite for comprehensive web security testing.

### Installation Instructions
1. Open Burp Suite.
2. Go to the BApp Store.
3. Search for "**Param Miner**" and click "**Install**".

### Usage
Once installed, Param Miner will **automatically scan requests** for hidden parameters and potential parameter pollution issues. You can customize its settings and view results in the Burp Suite interface.

## HTTP Parameter Pollution

>HTTP Parameter Pollution (HPP) occurs when an application processes multiple instances of the same parameter, potentially leading to unexpected behavior or security vulnerabilities. Special characters, such as Unicode characters, can sometimes be used to exploit these vulnerabilities.

### Example
Using the Unicode character for a pile of poo 💩 in parameters to test for HPP:
```bash
curl "https://example.com/page?param1=value1&param2=💩"
```
This can sometimes break applications or cause unexpected behavior, making it a useful test for robustness and security.