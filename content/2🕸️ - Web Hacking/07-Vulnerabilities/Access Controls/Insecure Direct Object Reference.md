# Basics

>[!What is an Insecure Direct Object Reference ?]
>Insecure direct object references (IDOR) are a type of access control vulnerability that arises when an application uses user-supplied input to access objects directly. The term IDOR was popularized by its appearance in the OWASP 2007 Top Ten. However, it is just one example of many access control implementation mistakes that can lead to access controls being circumvented. IDOR vulnerabilities are most commonly associated with horizontal privilege escalation, but they can also arise in relation to vertical privilege escalation.

![IDOR - securityzines](https://miro.medium.com/v2/resize:fit:1400/0*2SG8W4aDM_Ii6Aws)

## Examples

### Horizontal Privilege Escalation
Consider a website that uses the following URL to access the customer account page, by retrieving information from the back-end database:

`https://insecure-website.com/customer_account?id=132355`

Here, the **customer number is used directly as a record index** in queries that are performed on the back-end database. If no other controls are in place, an attacker can simply modify the `id` value, bypassing access controls to view the records of other customers. This is an example of an IDOR vulnerability leading to horizontal privilege escalation.

### With Cookies

Consider a website that stores sessions cookies like the following, with an incremental number for each user. An attacker can modify the value of the `user_id` cookie to gain access to other users account or an Administrator account.
![[Pasted image 20240906144247.png]]

### Sensitive information retrieval

Consider a website might save chat message transcripts to disk using an incrementing filename, and allow users to retrieve these by visiting a URL like the following:

`https://insecure-website.com/static/12144.txt`

In this situation, an attacker can simply modify the filename to retrieve a transcript created by another user and potentially obtain user credentials and other sensitive data.