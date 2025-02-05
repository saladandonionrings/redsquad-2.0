
![[Pasted image 20240906164504.png]]

> Copyright to REDSQUAD

# Prerequisites

```bash
# get the iotgoat .img for static analysis and .vmdk for dynamic analysis (run it locally)
https://github.com/OWASP/IoTGoat/releases

# packets
apt update
apt install binwalk
apt install squashfs-tools 

# TOOLS
# firmwalker
mkdir tools
cd tools
git clone https://github.com/scriptingxss/firmwalker.git

# testssl
git clone https://github.com/drwetter/testssl.sh.git

# linpeas
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
```

# Static Analysis

**Get the IOTGoat .img file.**

## Binwalk

```bash
# Scan to identify code, files and other information
binwalk IOTGoat.img

# Recursively extract the firmware and decompress the file
binwalk -reM IOTGoat.img
```

_Squashfs filesystem, little endian, version 4.0, compression:xz_

```bash
# extract the .img file and save it as a .bin file 
dd if=IoTGoat-raspberry-pi2.img bs=1 skip=29360128 of=iotgoat.bin
```

**.bin analysis :**

![[Pasted image 20240906164725.png]]

It is a squashfs file system so we can use **unsquashfs**, which will allow us to unpack the squashfs file system:
```bash
unsquashfs iotgoat.bin
```

## Vulnerabilities
### No 1. Weak, Guessable or Harcoded Passwords

> Use of easily found, publicly available, or unmodifiable credentials, including backdoors in firmware or client software that allow unauthorized access to deployed systems.

**Firmwalker**

>[!INFO] This tool allows us to search the extracted firmware file system for **juicy elements** (_passwords, keys, info leak, etc_).

```bash
./firmwalker.sh ../firmware/_IoTGoat.img.extracted/squashfs-root/ ./IoTGoat.txt
```

Excerpt :
![[Pasted image 20240906164747.png]]

```bash
# in the squashfs folder of the unpacked firmware
cat etc/shadow

# output :
root:$1$Jl7H1VOG$Wgw2F/C.nLNTC.4pwDa4H1:18145:0:99999:7:::
daemon:*:0:0:99999:7:::
ftp:*:0:0:99999:7:::
network:*:0:0:99999:7:::
nobody:*:0:0:99999:7:::
dnsmasq:x:0:0:99999:7:::
dnsmasq:x:0:0:99999:7:::
iotgoatuser:$1$79bz0K8z$Ii6Q/if83F1QodGmkb4Ah.:18145:0:99999:7:::
```

We have 2 password hashes: root and a user **iotgoatuser**.

### **Bruteforce SSH**

With the user found, we try a **bruteforce** attack on the ssh service (port 22).\
We use the **mirai-botnet** wordlist of **SecLists**, because it is using a wordlist with the common default IoT devices passwords.

```bash
# We want to filter only on the passwords because the list is of form user:password
awk '{print $2}' /usr/share/SecLists/Passwords/Malware/mirai-botnet.txt > /usr/share/SecLists/Passwords/Malware/mirai_pass.txt

# Bruteforce with Hydra
hydra -f -t 4 -l iotgoatuser -P /usr/share/SecLists/Passwords/Common-Credentials/mirai_pass.txt ssh://10.10.10.5
```

> **Credentials :**\
> iotgoatuser:7ujMko0vizxv

### No 6: Insufficient Privacy Protection

> User's personal information stored on the device or in the ecosystem that is used in an insecure, inappropriate or unauthorized manner.

Firmwalker found a database containing personal information, unsecured since it was stored locally:

![[Pasted image 20240906164758.png]]

This allowed us to extract unsecured sensitive information :

![[Pasted image 20240906164802.png]]

# Dynamic Analysis
## Vulnerabilities & Exploits
### No 2. Insecure Network Services

> Unnecessary or unsecured network services running on the device itself, especially those exposed to the Internet, can compromise the confidentiality, integrity/authenticity, or availability of information or allow an attacker to gain unauthorized remote control.

1. **Exposed services**

```bash
❯ nmap -A -Pn 10.10.10.5

PORT    STATE SERVICE        VERSION
22/tcp  open  ssh            
25/tcp  open  smtp			 
53/tcp  open  domain		
80/tcp  open  http			
110/tcp open  pop3			 
119/tcp open  nntp
143/tcp open  imap
443/tcp open  https
465/tcp open  smtps
563/tcp open  snews
587/tcp open  submission
993/tcp open  imaps
995/tcp open  pop3s
5000/tcp  open   upnp		 
5515/tcp  open   unknown
65534/tcp open	 unknown
```

Due to a lack of restriction in network filtering, some services are exposed on the Internet. This can potentially allow an attacker to identify vulnerabilities in services that are often more vulnerable.

**2. MiniUPnP 2.1**\
The MiniUPnP version is vulnerable to :

* **Use after free** vulnerability (_CVE-2019-12106_)
* **Information disclosure** vulnerability (_CVE-2019-12107_)
* Multiple **DoS** vulnerabilities due to NULL pointer dereferences (_CVE-2019-12108, CVE-2019-12109, CVE-2019-12110, CVE-2019-12111_)

**3. dnsmasq 2.73**

![[Pasted image 20240906164814.png]]

The version of dnsmasq is outdated and has 20 vulnerabilities.

Some **PoC** are available here : https://github.com/google/security-research-pocs/tree/master/vulnerabilities/dnsmasq

>[!WARNING]
**Unfortunately, we were unable to test these exploits because the virtual network interfaces had problems accessing IoTGoat and running the UPnP and Dnsmasq exploits.**

**4. DropBear 2017.75-7.1**\\

![[Pasted image 20240906164902.png]]

This version has 4 vulnerabilities.

Moreover, a configuration vulnerability is present in the configuration file: the **possibility to connect as root via ssh** is enabled:

![[Pasted image 20240906164921.png]]

### No 7: Insecure Data Transfer

> Lack of encryption or access control of sensitive data.

We use the **testssl.sh** tool, which allows us to check the service of a server on any port for support of TLS/SSL encryptions, protocols as well as recent cryptographic flaws and more.
The cipher suites used (_CBC_) by the web service are **obsolete** :

![[Pasted image 20240906164927.png]]

Also:

* The certificate does not match the URI provided,
* Certificate is **self-signed** (null trust chain)
* No CRL or OCSP URI provided

![[Pasted image 20240906164932.png]]

Moreover, the **HSTS** header is not implemented on the Web service and no security header is implemented:

![[Pasted image 20240906164942.png]]

The web service is potentially vulnerable to **Lucky13** :

![[Pasted image 20240906164950.png]]

**2.** We also found :
* The absence of the **X-Frame-Options** header
  * Could lead to a **ClickJacking** attack.
* Absence of **Anti-CSRF** token
  * Could lead to a **CSRF** attack via one of the forms.

Finally, **port 25 is open** on the machine, allowing the use of **telnet**, a non-secure communication protocol.

**🚪 Root BackDoor - PoC**

After having decrypted the password of the **iotgoatuser** user, we connect to the machine with ssh.\
Many manual tests have been done to try to **escalate privileges** and become root (_cf._ [_https://hackerbible.gitbook.io/en/pentest-linux/privilege-escalation/manual-checks_](https://hackerbible.gitbook.io/en/pentest-linux/privilege-escalation/manual-checks)).\
For example, we have run **Linpeas** on the machine in order to try to find points of privilege escalation attempts:

![[Pasted image 20240906165036.png]]

Linpeas allowed us to list the active ports on the machine, this allowed us to see 2 interesting ports open on the machine: **5515** and **65534**.

The port **65534** is open on the machine, we try to connect to it with netcat :

![[Pasted image 20240906165042.png]]

We were not able to crack the root password, so this backdoor is not useful because we can only connect as _iotgoatuser_.

Also, the port **5515** is open. We try to connect to it with netcat :

![[Pasted image 20240906165047.png]]

This reverse shell gives us access as a **root** so we can **change its password** in order to access the web interface:

![[Pasted image 20240906165051.png]]

> \*Since this is an OpenWRT router, the SSH password and the root web interface password are the same.

New root password on the web interface: **password**

### No 5. Use of Insecure or Outdated Components

> Use of outdated or insecure software components/libraries that could allow the device to be compromised. This includes insecure customization of operating system platforms and the use of third-party software or hardware components from a compromised supply chain.

**1. Busybox 1.28.4**

![[Pasted image 20240906165237.png]]

This version of Busybox has 14 exploits (_cf._ [_https://cyber.vumetric.com/vulns/busybox/busybox/1-28-4/_](https://cyber.vumetric.com/vulns/busybox/busybox/1-28-4/))

**2. Linux Kernel 4.14.95**

![[Pasted image 20240906165251.png]]

This version of Kernel **is** from **2017**. Thus, it has 25 exploits (_cf._ [_https://www.security-database.com/cpe.php?detail=cpe%3A2.3%3Ao%3Alinux%3Alinux\_kernel%3A4.14.95%3A\*%3A\*%3A\*%3A\*%3A\*%3A\*%3A\*_](https://www.security-database.com/cpe.php?detail=cpe%3A2.3%3Ao%3Alinux%3Alinux\_kernel%3A4.14.95%3A%2A%3A%2A%3A%2A%3A%2A%3A%2A%3A%2A%3A%2A))

**3. pppd version 2.4.7**

![[Pasted image 20240906165308.png]]

This version of pppd is vulnerable to a denial of service and arbitrary code execution attack(_cf._ [_https://www.cyberveille-sante.gouv.fr/cyberveille/1646_](https://www.cyberveille-sante.gouv.fr/cyberveille/1646)).

### No 4. Lack of Secure Update Mecanism

> Lack of security updates. This includes lack of firmware validation on the device, lack of secure encryption, lack of anti-rollback mechanisms, and lack of notifications of security changes due to updates.

* **OpenWRT Version :**\\

![[Pasted image 20240906165323.png]]

This version is **vulnerable** to **21 exploits**.

**CVE-2020-7982 - PoC**

https://forallsecure.com/blog/uncovering-openwrt-remote-code-execution-cve-2020-7982

### No 3. Insecure Ecosystem Interfaces

> Unsecured web interfaces, APIs, mobile devices in the ecosystem, can allow the device or its related components to be compromised. The most common issues are lack of authentication/authorization, lack of or weak encryption, and lack of input and output filtering.

**CVE-2019-18992 - PoC**

> OpenWrt 18.06.2 is vulnerable to a **stored XSS attack** via these fields at the URI _/cgi-bin/luci/admin/network/firewall/rules_: "**Open ports on router**", "**New forward rule**" and "**New Source NAT**".

An XSS payload has been inserted in _/cgi-bin/luci/admin/network/firewall/rules_, in the **New Forward Rule** field.

![[Pasted image 20240906165406.png]]

Then we click on _Edit_ to trigger the XSS :\\

![[Pasted image 20240906165409.png]]

This XSS is also present in the **New Forward Rule** and **New Source Nat** fields, as well as in **Traffic Rules Name**.

**CVE-2019-18993 - PoC**

> OpenWrt 18.06.4 is vulnerable to **XSS attack stored** via the "New port forward" field at the URI /cgi-bin/luci/admin/network/firewall/forwards

This XSS payload has been inserted in _/cgi-bin/luci/admin/network/firewall/forwards_, in the **New Port Forward** field:

```html
<script>alert("HACKED!");</script>
```

![[Pasted image 20240906165416.png]]

By clicking on _Edit_, we trigger the XSS :

![[Pasted image 20240906165420.png]]

**CVE-2019-25015 - PoC**

> LuCI in OpenWrt versions **18.06.0 to 18.06.4** contains a **XSS vulnerability stored** via a modified SSID.

An XSS payload was inserted in _/cgi-bin/luci/admin/network/wireless/wl0.network1_ in the **ESSID** field.

![[Pasted image 20240906165433.png]]

Then we click on **Save & Apply,** to trigger the XSS :

![[Pasted image 20240906165436.png]]

### Lack of Anti Bruteforce Mechanism

We were able to test a brute force attack on the web application folders with the BurpSuite tool. This one does not implement any anti-bruteforce mechanism.

![[Pasted image 20240906165440.png]]

**Command Execution**

There is a page _cgi-bin/luci/admin/iotgoat_ on the web service :

![[Pasted image 20240906165444.png]]

At the root of **/iotgoat**, we find this _hidden_ page:

![[Pasted image 20240906165450.png]]

This is a **Command Execution** vulnerability, allowing us to access the ash shell as **root** :

![[Pasted image 20240906165512.png]]

From there, an attacker might be able to **take full control** of the machine.

**DOS - CVE-2019-19945 - PoC**

> uhttpd in OpenWrt versions up to 18.06.5 and 19.x up to 19.07.0-rc2 has an integer signature error. This leads to an out-of-bounds access to a heap buffer and a subsequent crash. It can be triggered by an HTTP POST request to a CGI script, specifying both "Transfer-Encoding: chunked" and a large negative Content-Length value.

https://github.com/mclab-hbrs/openwrt-dos-poc

### No 8. Lack of Device Management

> Lack of security support on production-deployed devices, including asset management, update management, secure decommissioning, system monitoring and response capabilities.

Logs are not enabled :

![[Pasted image 20240906165534.png]]

In addition, OpenWRT packages are **not updated by default.**

### No 9: Insecure Default Settings

> Devices or systems are shipped with unsecured default settings or lack the ability to make the system more secure by preventing operators from changing configurations.

* Using the default **root** user to log in to the web interface
* UPnp enabled by default and without _secure mode_

![[Pasted image 20240906165542.png]]
