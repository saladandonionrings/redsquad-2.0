# Overview

> [!INFO]
> This tool adds **SOCKS** proxy functionality to **Terminal Services** (or Remote Desktop Services) and **Citrix** (XenApp/XenDesktop).
> It uses a dynamic virtual channel that allows us to communicate via an open RDP/Citrix connection without the need to open a new socket, connection or port on a firewall.

https://github.com/nccgroup/SocksOverRDP

## Resources

https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding#socksoverrdp-and-proxifier

[Partial Demo, Youtube](https://www.youtube.com/watch?t=896&v=ZD1mQHZKymQ&feature=youtu.be)

## Servers / Requirements

* 2 Windows machines (Server or Workstation running)
* Linux Host

### PoC

#### Machines

* **Windows 2008 (Server)**:
  * 2 network cards: 192.168.56.127 (private host) and 10.0.2.4 (NAT network)
  * RDP and IIS roles enabled
  * IIS server on NAT network 10.0.2.4 (inaccessible by Client machine and Linux Host)
* **Windows 10 (Client)**:
  * 1 network card (private host): 192.168.56.126
  * RDP enabled
* **Linux host**
  * 192.168.56.1
* **On the 2 Windows workstations** :
  * [Installer Visual C++ Redistributable for Visual Studio 2015 package](https://learn.microsoft.com/fr-fr/cpp/windows/latest-supported-vc-redist?view=msvc-170)

#### Steps

1. Install [Releases](https://github.com/nccgroup/SocksOverRDP/releases/tag/v1.0) : .DLL on the Client and .EXE on the Server
2. On **Client**, administrator command prompt:

```powershell
# register the SocksOverRDP-Plugin.dll file as a COM component in Windows so that it works with RDP
regsvr32.exe .\SocksOverRDP-Plugin.dll
```

3. On **Client**: connect to the server via RDP: `mstsc.exe`.
   1. Run a powershell command (admin) on the **server via RDP** :

```powershell
# tunnelling of SOCKS connection via RDP, server listens to incoming RDP connections
# verbose option
.\SocksOverRDPServer.exe -v
```

4. Back to **Client**, administrator command prompt:

```powershell
# list open ports
netstat -anot | findstr LISTEN
# see : TCP 127.0.0.1:1080
```

**Add SOCKS proxy configuration**

`Control Panel > Network & Internet > Internet Options > Connections > LAN Settings > Use Proxy Server (Advanced) > Socks : 127.0.0.1:1080 Proxy Settings > Use a proxy server (ON) > Address: http://socks=127.0.0.1 ; Port: 1080`

(Or use proxifier: https://www.proxifier.com/)

5. **Access: http://10.0.2.4** via Client machine

#### SOCKS Chain

>[!HINT]
>We want to :
From Linux machine (192.168.56.0/24) -> access server network
Chain : **Server -> Client -> Linux**

**Client**
_(After step 2)_
1. By default, the binding is set to **127.0.0.1** on the Client machine (inaccessible by the Linux machine), so change this value in **regedit.exe** to its **private IP address** (192.168.56.126): `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Terminal Server Client\Default\AddIns\SocksOverRDP-Plugin`.
2. _Repeat steps 3 to 4 to apply this change_

**Linux**
1. Install **proxychains4** on the Linux machine:

```bash
sudo apt-get update sudo apt-get install proxychains4
```

2. Modify the **proxychains4** configuration file and configure it to forward traffic to the SOCKS proxy created by SocksOverRDP on the Windows client:

```bash
# /etc/proxychains4.conf or /etc/proxychains.conf
socks5 192.168.56.126 1080 # ip client windows + port socks
```

3. Test the connection to the IIS server from the Linux machine:

```bash
proxychains4 nc -znv 10.0.2.4 80
# works!
```
