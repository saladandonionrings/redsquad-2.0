# Vulnerabilities
## 1. Weak, Guessable or Harcoded Passwords

[Check IOT default password](https://defpass.com/index.php)

Use of :
* Easily bruteforced
* Publicly available
* Unchangeable credentials

Including backdoors in firmware or client software that grants unauthorized access.

## 2. Insecure Network Services

- Unneeded or insecure network services running on the device itself, especially:
	* Those **exposed** to the Internet
	* Any that compromise the confidentiality, integrity/authenticity, or availability of information
	* Any service that allows unauthorized remote control

## 3. Insecure Ecosystem Interfaces

See OWASP TOP 10, insecure interfaces in the ecosystem outside the device :
* Web
* Backend API
* Cloud
* Mobile

**Common issues :**
* Lack of authentication
* Lack of authorization
* Lacking or weak encryption
* Lack of input and output filtering

## 4. Lack of Secure Update Mechanism

- Lack of ability to securely update the device.
	* Lack of firmware validation on device
	* Lack of secure delivery (un-encrypted in transit)
	* Lack of anti-rollback mechanisms
	* Lack of notifications of security changes due to updates

## 5. Use of Insecure or Outdated Components

- Use of deprecated or insecure software components/libraries that could allow the device to be compromised.
	* Insecure customization of operating system platforms
	* Third-party software libraries from a compromised supply chain
	* Third-party hardware components from a compromised supply chain
- **Examples :**
	- HeartBleed
	- Spectre
	- Meltdown

## 6. Insufficient Privacy Protection

- User’s personal information stored on the device or in the ecosystem that is used insecurely, improperly, or without permission.
- **Examples :** 
	- Location
	- Emails
	- Addresses

## 7. Insecure Data Transfer and Storage

- Lack of encryption or access control of sensitive data anywhere within the ecosystem, including at rest, in transit, or during processing.
- **Examples :** lack of HSTS, no disk encryption

## 8. Lack of Device Management

- **Examples :**
	- No update mechanism
	- No logging

## 9. Insecure Default Settings

- Bad filesystem permissions
- Exposed services running as root

## 10. Lack of Physical Hardening

- Easily Available Debug Port Discovery

# Resources

[IOT Device Pentest - OWASP](https://owasp.org/www-chapter-pune/meetups/2019/August/IoT_Device_Pentest_by_Shubham_Chougule.pdf)
[Checklist](https://github.com/OWASP/owasp-istg/blob/main/checklists/checklist.md)
[OWASP IOT SECURITY TESTING GUIDE](https://owasp.org/owasp-istg/index.html)

## Tools

[HomePWN tool](https://github.com/Telefonica/HomePWN)

