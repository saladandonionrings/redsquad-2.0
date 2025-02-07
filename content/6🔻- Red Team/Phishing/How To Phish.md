>[!WARNING] DISCLAIMER
>The information provided in this post is for **educational purposes only** and aims to inform readers about phishing techniques for awareness and defense purposes. 
>Any mention of phishing tools or methods is **not** an endorsement or encouragement of illegal activities.  
>Readers are strongly advised to use this knowledge responsibly, ensuring compliance with all relevant laws and ethical standards.  
>The blog owner assumes **no responsibility** for any unlawful or harmful actions resulting from the use of this information.
# Find a domain name
>Create a domain name that looks like the target's domain.
- https://github.com/elceef/dnstwist

```bash
git clone https://github.com/elceef/dnstwist.git
cd dnstwist
pip install .

# usage, find domains
dnstwist $domain
```

# Find a person to impersonate

**OSINT :**
* [hunter.io](https://hunter.io)
* [LinkedIn](https://www.linkedin.com/home/?originalSubdomain=fr)

- SMTP User Enumeration : https://pentestmonkey.net/tools/user-enumeration/smtp-user-enum

```bash
smtp-user-enum -U /usr/share/seclists/Usernames/top-usernames-shortlist.txt "10.129.87.12" "25" -m RCPT
smtp-user-enum -U ./test-users.txt "10.129.87.12" "25" -m RCPT
```

# Resources

https://github.com/PhishyAlice/awesome-phishing
