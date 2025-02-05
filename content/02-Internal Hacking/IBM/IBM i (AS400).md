# Overview
- [IBM i (AS400) - Security Reference](https://www.ibm.com/docs/en/ssw_ibm_i_73/pdf/sc415302.pdf)

# MindMap
![[AS400-audit.jpg]]
# Ports Scanning
## Common non-secure

| Service Name  | Description                                                                                              | Port Number |
| ------------- | -------------------------------------------------------------------------------------------------------- | ----------- |
| ddm           | DDM server is used to access data via DRDA and for record level access.                                  | 446         |
| As-svrmap     | Port mapper returns the port number for the requested server.                                            | 449         |
| As-admin-http | HTTP server administration                                                                               | 2001        |
| As-mtgctrlj   | Management Central server is used to manage multiple AS/400s in a network.                               | 5544        |
| As-mtgctrl    | Management Central server is used to manage multiple AS/400s in a network                                | 5555        |
| As-central    | Central server is used when a Client Access license is required and for downloading translation tables.  | 8470        |
| As-database   | Database server is used for accessing the AS/400 database.                                               | 8471        |
| As-dtaq       | Data Queue server allows access to the AS/400 data queues, used for passing data between applications.   | 8472        |
| As-file       | File Server is used for accessing any part of the AS/400 file system                                     | 8473        |
| as-netprt     | Printer Server is used to access printers known to the AS/400.                                           | 8474        |
| as-rmtcmd     | Remote command server is used to send commands from a PC to an AS/400 and for program calls              | 8475        |
| as-signon     | Sign-on server is used for every Client Access connection to authenticate users and to change passwords. | 8476        |
| as-usf        | Ultimedia facilities are used for multimedia data.                                                       | 8480        |
## Common secure (SSL)

| Service name   | Description                                                                                              | Port Number |
| -------------- | -------------------------------------------------------------------------------------------------------- | ----------- |
| ddm-ssl        | DDM server is used to access data via DRDA and for record level access.                                  | 447, 448    |
| telnet-ssl     | Telnet server.                                                                                           | 992         |
| as-admin-https | HTTP server administration.                                                                              | 2010        |
| as-mgtctrl-ss  | Management Central server is used to manage multiple AS/400s in a network.                               | 5566        |
| as-mgtctrl-cs  | Management Central server is used to manage multiple AS/400s in a network.                               | 5577        |
| as-central-s   | Central server is used when a Client Access license is required and for downloading translation tables.  | 9470        |
| as-database-s  | Database server is used for accessing the AS/400 database.                                               | 9471        |
| as-dtaq-s      | Data Queue server allows access to the AS/400 data queues, used for passing data between applications.   | 9472        |
| as-file-s      | File Server is used for accessing any part of the AS/400 file system.                                    | 9473        |
| as-netptr-s    | Printer Server is used to access printers known to the AS/400.                                           | 9474        |
| as-rmtcmd-s    | Remote command server is used to send commands from a PC to an AS/400 and for program calls.             | 9475        |
| As-signon-s    | Sign-on server is used for every Client Access connection to authenticate users and to change passwords. | 9476        |
## Scanning
```bash
nc -v -z -w 1 as400.victim.com 1-100 | grep "open"
```

# Configuration
## Retrieve all configuration
```bash
# General infos
WRKSYSVAL SYSVAL(*ALL) OUTPUT(*PRINT)

# Security infos
WRKSYSVAL SYSVAL(*SEC) OUTPUT(*PRINT)

# Retrieve *PUBLIC Rights
PRTPUBAUT OBJTYPE(*USRPRF)

# Retrieve all rights on accounts
PRTPVTAUT OBJTYPE(*USRPRF)

# Audit one account
DSPUSRPRF USRPRF(LOGIN) OUTPUT(*OUTFILE) OUTFILE(COGICEO/TEST)
# Take the file : right click > export > file

# Trivial passwords ?
ANZDFTPWD

# Last passwords change 
PRTUSRPRF TYPE(*PWDINFO) SELECT(*SPCAUT) SPCAUT(*ALL)

# High privileges accounts - sort by privileges
PRTUSRPRF TYPE(*AUTINFO) SELECT(*SPCAUT) SPCAUT(*SECADM)
PRTUSRPRF TYPE(*AUTINFO) SELECT(*SPCAUT) SPCAUT(*ALLOBJ)
PRTUSRPRF TYPE(*AUTINFO) SELECT(*SPCAUT) SPCAUT(*JOBCTL)

# High privileges accounts - sort by classes
PRTUSRPRF TYPE(*AUTINFO) SELECT(*USRCLS) USRCLS(*SYSOPR)
PRTUSRPRF TYPE(*AUTINFO) SELECT(*USRCLS) USRCLS(*PGMR)
PRTUSRPRF TYPE(*AUTINFO) SELECT(*USRCLS) USRCLS(*SECADM)
PRTUSRPRF TYPE(*AUTINFO) SELECT(*USRCLS) USRCLS(*SECOFR)

```

## Users Passwords
```bash
rexec -d -l LOGIN -p PASSWORD -b IP "ANZDFTPWD ACTION(*NONE)"
```

## Retrieve libraries
```bash
rexec -l LOGIN -p PASSWORD -b IP "QSH CMD('ls /qsys.lib/')" | strings | grep -E '*.LIB$' | awk -F"." '{print $1}' > lib

```

## How to get & crack AS/400 hashes?

**Access to an AS/400 server with `*ALLOBJ` and `*SECADM` privileges**

> Depending on the current setting of the QPWDLVL system value, password hashes are stored in different formats on the AS/400:
>
> * **QPWDLVL 0**: IBM DES hashes (supported by JtR with our 'as400-des' format plugin) LM hashes (supported by default by JtR) SHA1 uppercase hashes (supported by JtR with our 'as400-ssha1' format plugin)
> * **QPWDLVL 1**: IBM DES hashes (supported by JtR with our 'as400-des' plugin) SHA1 uppercase hashes (supported by JtR with our 'as400-ssha1' plugin)
> * **QPWDLVL 2**: IBM DES hashes\* (supported by JtR with our 'as400-des' plugin) LM hashes\*\* (supported by default by JtR) SHA1 uppercase hashes\*\*\* (supported by JtR with our 'as400-ssha1' plugin) SHA1 mixed case hashes (supported by JtR with our 'as400-ssha1' plugin)
> * **QPWDLVL 3**: SHA1 uppercase hashes\*\*\* (supported by JtR with our 'as400-ssha1' plugin) SHA1 mixed case hashes (supported by JtR with our 'as400-ssha1' plugin)

> * Only if QPWDMAXLEN <=10
>   * Only if QPWDMAXLEN <=14
>   * Depending on password policy configuration

### PREREQUISITES

* The latest version of **IBMiScanner** (part of **hack400tool**), available on https://github.com/hackthelegacy/hack400tool
* The latest **john the ripper** jumbo release, including 'as400-des' and 'as400-ssha1' plugins.

**LM hashes**

* Open IBMiScanner tool
* Connect
* From the list of available scans, select option **26: 'SECURITY: Get John the Ripper hashes (LM hash)**'
* In the **output** directory, a file named '**lmhashes.txt**' will be created.
* Copy the file to your John the Ripper 'run' directory
* Run john the ripper: john --format=LM {filename} 
* Enjoy the passwords :)

**DES hashes**

* Open IBMiScanner tool
* Connect
* From the list of available scans, select option **29: 'SECURITY: Get John the Ripper hashes (DES)'**
* In the **output** directory, a file named '**DES-hashes.txt**' will be created.
* Copy the file to your John the Ripper 'run' directory
* Run john the ripper: john --format=as400-des {filename} 
* Enjoy the passwords :)

**SHA-1 hashes**

(Please note that this method is generic for both mixed and upper case)

* Open IBMiScanner tool
* Connect
* From the list of available scans, select option **27: 'SECURITY: Get John the Ripper hashes (SHA-1 hash uppercase)'** for uppercase hashes.
* For mixed case hashes, you can choose for option **28: 'SECURITY: Get John the Ripper hashes SHA-1 hash mixed case)'** respectively.
* In the **output** directory, a file named '**SHA-uc-hashes.txt**' for uppercase hashes or 'SHA-mc-hashes.txt' for mixed case hashes will be created.
* Copy the file to your John the Ripper 'run' directory
* Run john the ripper: john --format=as400-ssha1 {filename} 
* Enjoy the passwords :)

> Note: In case you used an older version of the IBMiscanner tool that outputs hashes in the format **userid:hash**, you can use the ibmiscanner2john.py script to convert the file into a format that can be processed by JtR



# Pentest

## Tools

```bash
git clone https://github.com/hackthelegacy/hack400tool
```

- IBM System i Access V6R1_64
- IBM System i Access V5R1 32b
- Reflexion Client TN5250

## Privilege Escalation

- https://blog.silentsignal.eu/2022/09/05/simple-ibm-i-as-400-hacking/