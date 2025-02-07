# HTTP Security Headers Overview

HTTP security headers help improve the security of web applications by enforcing various security policies directly within the browser. These headers prevent a wide range of attacks, including cross-site scripting (XSS), clickjacking, and protocol downgrade attacks.

## HTTP Strict-Transport-Security (HSTS)

HSTS ensures that browsers only access a site using HTTPS, preventing protocol downgrade attacks and cookie hijacking.

### Syntax
```text
Strict-Transport-Security: max-age=<expire-time>
Strict-Transport-Security: max-age=<expire-time>; includeSubDomains
Strict-Transport-Security: max-age=<expire-time>; preload
```

### Directives
- `max-age=<expire-time>`: Specifies the time, in seconds, that the browser should remember to only access the site via HTTPS.
- `includeSubDomains`: Applies the rule to all subdomains if specified.
- `preload`: Allows the site to be included in browsers' HSTS preload lists. 

## Content Security Policy (CSP)

CSP helps prevent cross-site scripting (XSS) and other cross-site injections by allowing site administrators to control resources the user agent is allowed to load.

### Example
```text
Content-Security-Policy: script-src 'self'
```

### Google CSP Scanner
[Google CSP Evaluator](https://csp-evaluator.withgoogle.com/)

## X-Frame-Options

This header improves protection against clickjacking attacks by controlling whether the content can be displayed in a frame.

### Values
- `deny`: Prevents any framing.
- `sameorigin`: Allows framing from the same origin.
- `allow-from URI`: Allows framing from the specified URI.

### Example
```text
X-Frame-Options: deny
```

## Referrer-Policy

The Referrer-Policy header governs which referrer information is included with requests made.

### Values
- `no-referrer`
- `no-referrer-when-downgrade`
- `origin`
- `origin-when-cross-origin`
- `same-origin`
- `strict-origin`
- `strict-origin-when-cross-origin`
- `unsafe-url`

### Example
```text
Referrer-Policy: no-referrer
Referrer-Policy: origin
```

## X-Content-Type-Options

This header prevents the browser from interpreting files as a different MIME type than what is declared by the content type in the HTTP headers.

### Example
```text
X-Content-Type-Options: nosniff
```

## Obsolete Headers

### HTTP Public Key Pinning (HPKP)
HPKP is no longer recommended and may be removed from web standards.

### X-XSS-Protection
X-XSS-Protection is also no longer recommended and should be set to `0` to disable it.
```text
X-XSS-Protection: 0
```
Source: [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection)

# NextGen Headers Scanner

NextGen Headers Scanner is a tool to scan and evaluate the security headers of a given target URL.

## Installation and Usage
```bash
# Clone the repository
git clone https://github.com/saladandonionrings/NextGen-HeadersScanner.git

# Install the required dependencies
pip install -r requirements.txt

# Run the scanner
python h_scan -u $target_url
```
Replace `$target_url` with the URL of the target you want to scan.

By implementing and properly configuring these HTTP security headers, web administrators can significantly enhance the security of their web applications.