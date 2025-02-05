# 🔬 Volatility

> Volatility is a tool that can be used to analyze a volatile memory of a system. You can inspect processes, look at command history, and even pull files and passwords from a system without even being on the system.

#### Installation

```bash
# volatility3
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3
python3 setup.py install
python3 vol.py —h

# volatility2
# Download the executable from https://www.volatilityfoundation.org/26
git clone https://github.com/volatilityfoundation/volatility.git
cd volatility
python setup.py install
```

#### Useful commands

```bash
# image infos
volatility -f file.mem imageinfo

# Hive and Registry key values
volatility -f file.mem --profile=MyProfile hivelist
volatility -f file.mem --profile=MyProfile printkey -K "MyPath"

# Analyzing processes
volatility -f file.mem --profile=Win7SP1x64 pslist
# list parent-child relations processes
volatility -f file.mem --profile=Win7SP1x64 pstree

# list all app running
volatility-f file.mem --profile=Win7SP1x64 shimcache > shimcache.txt

# analyze network connections
volatility -f file.mem --profile=Win7SP1x64 netscan > output_netscan.txt
# running sockets & open connections
volatility -f file.mem --profile=Win7SP1x64 connscan
volatility -f file.mem --profile=Win7SP1x64 sockets

# commandline history
volatility -f file.mem --profile=Win7SP1x64 cmdline
volatility -f file.mem --profile=Win7SP1x64 consoles
```

#### Detect malicious files

In volatility, there exists an attribute named malfind. This is actually an inbuilt plugin and can be used for malicious process detection.

```bash
volatility -f file.mem --profile=Win7SP1x64 -D <Output_Location> -p $PID malfind
# dump infected process
volatility -f file.mem --profile=Win7SP1x64 procdump -p 3496 --dump-dir $dumpfolder
```
