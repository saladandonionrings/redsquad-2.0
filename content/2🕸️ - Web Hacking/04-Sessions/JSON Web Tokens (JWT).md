# Brief

JSON web tokens (JWTs) are a standardized format for sending cryptographically signed JSON data between systems. They can theoretically contain any kind of data, but are most commonly used to send information ("claims") about users as part of authentication, session handling, and access control mechanisms.

Unlike with classic session tokens, all of the data that a server needs is stored client-side within the JWT itself. This makes JWTs a popular choice for highly distributed websites where users need to interact seamlessly with multiple back-end servers.

A JWT consists of **3 parts**: a header, a payload, and a signature. These are each separated by a dot, as shown in the following example:

![[Pasted image 20241024152544.png]]

The header and payload parts of a JWT are just **base64url-encoded** JSON objects. The header contains metadata about the token itself, while the payload contains the actual "claims" about the user. For example, you can decode the payload from the token above to reveal the following claims:

```json
{
    "iss": "portswigger",
    "exp": 1648037164,
    "name": "Carlos Montoya",
    "sub": "carlos",
    "role": "blog_author",
    "email": "carlos@carlos-montoya.net",
    "iat": 1516239022
}
```

In most cases, this data can be easily read or modified by anyone with access to the token. Therefore, the security of any JWT-based mechanism is heavily reliant on the **cryptographic signature**.

## Signature part

The server that issues the token typically generates the signature by **hashing the header** and **payload**. In some cases, they also encrypt the resulting hash. Either way, this process involves a **secret signing key**. This mechanism provides a way for servers to verify that none of the data within the token has been tampered with since it was issued:

* As the signature is directly derived from the rest of the token, changing a single byte of the header or payload results in a mismatched signature.
* Without knowing the server's secret signing key, it shouldn't be possible to generate the correct signature for a given header or payload.

## Particularities

> In other words, a JWT is usually either a JWS or JWE token. When people use the term "JWT", they almost always mean a JWS token. JWEs are very similar, except that the actual contents of the token are encrypted rather than just encoded.

# Attacking JWT

## Aim

JWT attacks involve a user sending modified JWTs to the server in order to achieve a malicious goal. Typically, this goal is to **bypass authentication** and **access controls** by impersonating another user who has already been authenticated.

The **impact** of JWT attacks is usually **severe**. If an attacker is able to create their own valid tokens with arbitrary values, they may be able to escalate their own privileges or impersonate other users, taking full control of their accounts.

### Flawed Signature Verification

For example, consider a JWT containing the following claims:

```json
{
    "username": "carlos",
    "isAdmin": false
}
```

**Changing the parameter "isAdmin" to true : Privilege Escalation.**

JWT libraries typically provide one method for verifying tokens and another that just decodes them. For example, the Node.js library jsonwebtoken has **verify()** and **decode()**. **Occasionally, developers confuse these two methods and only pass incoming tokens to the decode() method. This effectively means that the application doesn't verify the signature at all.**

### Accepting Tokens with No Signature

```json
{
    "alg": "HS256",
    "typ": "JWT"
}
```

**Change the "alg" value to "none". Remove the signature part but leave the trailing dot ".".**

> Even if the token is unsigned, the payload part must still be terminated with a trailing dot.

### Brute-forcing Secret Keys

Some signing algorithms, such as **HS256** (HMAC + SHA-256), use an arbitrary, standalone string as the secret key. Just like a password, it's crucial that this secret can't be easily guessed or brute-forced by an attacker. Otherwise, they may be able to create JWTs with any header and payload values they like, then use the key to re-sign the token with a valid signature.

When implementing JWT applications, **developers sometimes make mistakes** like forgetting to change default or placeholder secrets. They may even copy and paste code snippets they find online, then forget to change a hardcoded secret that's provided as an example. In this case, it can be trivial for an attacker to brute-force a server's secret using a **wordlist of well-known secrets**.

> JWT Wordlist : https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list$

**If the server uses an extremely weak secret, it may even be possible to brute-force this character-by-character rather than using a wordlist.**

#### How to

```bash
hashcat -a 0 -m 16500 $JWT $wordlist
# --show to output the result if you already run it
```

**Then :** Generate another key using JWT Editor Keys on BurpSuite, change the "k" parameter to the base64-encoded secret. Start accessing admin panels.

### Injecting self-signed JWTs via the JWK parameter
- `jwk` (JSON Web Key) - Provides an embedded JSON object representing the key.
- `jku` (JSON Web Key Set URL) - Provides a URL from which servers can fetch a set of keys containing the correct key.
- `kid` (Key ID) - Provides an ID that servers can use to identify the correct key in cases where there are multiple keys to choose from. Depending on the format of the key, this may have a matching `kid` parameter.

Example of this in the JWT header : 

```json
{
	"kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
	"typ": "JWT",
	"alg": "RS256",
	"jwk": {
		"kty": "RSA",
		"e": "AQAB",
		"kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
		"n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m"
	}
}
```

Sometimes, misconfigured servers **use any key that's embedded in the jwk parameter** to verify JWT signatures.

#### Exploit
Burp, the [JWT Editor extension](https://portswigger.net/bappstore/26aaa5ded2f74beea19e2ed8345a93dd) provides a useful feature to help you test for this vulnerability:

1. With the extension loaded, in Burp's main tab bar, go to the **JWT Editor Keys** tab.
2. [Generate a new RSA key.](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts#adding-a-jwt-signing-key)
3. Send a request containing a JWT to Burp Repeater.
4. In the message editor, switch to the extension-generated **JSON Web Token** tab and [modify](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts#editing-jwts) the token's payload however you like.
5. Click **Attack**, then select **Embedded JWK**. When prompted, select your newly generated RSA key.
6. Send the request to test how the server responds.

We have a JWT for our session and we want to impersonate another user : 

```json
eyJraWQiOiI1YjVjN2E3Zi1lNjFjLTQyNjAtODY5Ny04ZDY0NmZmOTI2NWQiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTcyOTc4MDUyNSwic3ViIjoid2llbmVyIn0.kbVtgHjMvPEm_iQbtOt6NBYX-4FGSGC21TP1aYAKcewtKRrIUm_edfl_dHC4w0D2vU26Z3_cnvDqazdTnBuuUaj69_C16_RNT9ZdHlWUIpM18KlTpkJEdzAX9JwElXC3rYYCN87pVHMx3Jxy1U8xiYN9MQ7tUhUidbHnRojE4RuiaGtjWyDw60Jht8HTYy6oLBI41e21NX9BvIyfsf_1ALORAR58H5nEV2FkXFJ7FN-iXbECZRDjj3ivqW5DNoqDEgbA_7u9g9QKOv1xVc9D3QCFut2--6O4pL2NwkV8hP66hvwsffH6jsVwbzEUj2nFS5RUn-sHdOlH2fx6HsWXAQ
```

Structure :
```
Token header values:
[+] kid = "5b5c7a7f-e61c-4260-8697-8d646ff9265d"
[+] alg = "RS256"

Token payload values:
[+] iss = "portswigger"
[+] exp = 1729780525    ==> TIMESTAMP = 2024-10-24 16:35:25 (UTC)
[+] sub = "wiener"
```

- We change the value of **sub** to **administrator**
- Attack > Embedded JWK
- Send Request

### Tools

#### JWT Toolkit V2
https://github.com/ticarpi/jwt_tool

```bash
git clone https://github.com/ticarpi/jwt_tool
cd jwt_tool/
python3 -m pip install termcolor cprint pycryptodomex requests
python3 jwt_tool.py $JWT

# Generate JWT with no algorithm (Alg:None)
python3 jwt_tool.py $JWT -a

# Null signature
python3 jwt_tool.py $JWT -n
```
