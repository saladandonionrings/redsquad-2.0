>[!What is ClickJacking ?]
>**Clickjacking**, also known as a “_UI redress attack_”, is when an attacker uses multiple transparent or opaque **layers** to trick a user into **clicking** on a **button** or **link** on another page when they were intending to click on the top level page.
Thus, the attacker is “**hijacking**” clicks meant for their page and routing them to another page, most likely owned by another application, domain, or both.

![GIF Clickjacking](https://geekflare.com/wp-content/uploads/2023/06/clickjacking1-google.gif)

## Check

* **No X-Frame-Options Header** with the content directive 'DENY'
* **No Content Security Policy** with the frame-ancestors directive 'none'

## Exploit

### Tools

[Using Burp to find Clickjacking vulnerabilities](https://portswigger.net/support/using-burp-to-find-clickjacking-vulnerabilities)
[ClickJackPoc - GitHub](https://github.com/Raiders0786/ClickjackPoc)

### Manual
Clickjacking attacks use CSS to create and manipulate layers. The attacker incorporates the target website as an iframe layer overlaid on the decoy website. An example using the style tag and parameters is as follows:

```html
<!-- copy in a form field -->
<iframe src="http://www.google.com" width="250" height="250"></iframe>
```
