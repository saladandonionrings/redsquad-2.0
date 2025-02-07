# Basics

>[!INFO] What is a CSRF attack ?
In a **C**ross-**S**ite **R**equest **F**orgery attack, a **malicious site tricks a victim's browser into making an unwanted request** to a different site on which the victim is authenticated, potentially causing the victim to perform an action on that site without their knowledge or consent.

![[Pasted image 20240906143834.png]]

## How does it work ?
_Imagine a scenario where you're logged into your online banking. While still logged in, you visit a different website that has some malicious code. This malicious website can send a request to your bank's website to transfer money without your knowledge. If the bank's website doesn't have proper CSRF protections, it would think that you made the request because your authentication cookies are automatically included by your browser._

## Key points about CCSRF :
1. **Relies on the authenticated state**: The attack works because browsers automatically include any cookies associated with a domain in requests made to that domain. So if you're authenticated to a website, that means any requests made to that website (even from a different website) will include your cookies.
2. **Doesn't steal data directly**: CSRF isn't about stealing data. Instead, it tricks the victim into performing actions without their knowledge.
3. **Exploits trusted relationships**: CSRF exploits the trust a website has in the user's browser, not necessarily a flaw in the website's design (although lack of CSRF protections is considered a design flaw).

## How to prevent CSRF attacks?

1. **Use Anti-CSRF Tokens**: The most common way to prevent CSRF attacks is to use anti-CSRF tokens. This involves sending a random token in each request which the server verifies. Since the malicious site won't know this token, it can't forge a valid request.
2. **SameSite Cookie Attribute**: Modern browsers support the `SameSite` cookie attribute, which can prevent the browser from sending cookies along with cross-site requests, mitigating the risk of CSRF attacks.
3. **Check the `Referer` and `Origin` Headers**: Servers can check these HTTP headers to see if a request is coming from a trusted origin.
4. **Require Reauthentication for Sensitive Actions**: For very sensitive operations, like changing a password, always prompt users to re-enter their current password.
5. **Be cautious with CORS**: Cross-Origin Resource Sharing (CORS) headers shouldn't be used recklessly, as they can allow unwanted cross-site interactions.

### CSRF Tokens weaknesses
#### Validation bypass
>Some applications skip the verification step if they can't find a token. If an attacker gains access to code containing a token, he can remove the token and successfully execute a CSRF attack. So, if a valid request to a server looks like this :

```http
POST /change_password
POST body:
password=pass123&csrf_token=93j9d8eckke20d433
```

An attacker need only remove the token and send it like this to execute the attack:

```http
POST /change_password
POST body:
password=pass123
```

#### Grouped tokens
>Some applications maintain a pool of tokens to validate user sessions, instead of designating a specific token for a session. All a hacker has to do is obtain one of the tokens already in the pool to impersonate any user on the site.

An attacker can connect to an application using its account to obtain a token, such as :
```http
[application_url].com?csrf_token=93j9d8eckke20d433
```
And since tokens are pooled, the attacker can copy and use that same token to log in to another user account, because you'll be using it again :

- **CSRFs can be tokens copied into the cookie** - Some applications copy the parameters linked to a token into the user's cookie. If an attacker gains access to this cookie, he can easily create another cookie, place it in a browser and execute a CSRF attack.

In this way, an attacker can log into an application using his account and open the cookie file to see the following:

```http
Csrf_token:93j9d8eckke20d433
```

He can then use this information to create another cookie to carry out the attack.

- **Invalid tokens** - Some applications do not match CSRF tokens to a user's session. In this case, an attacker can connect to a session, obtain a CSRF token similar to those mentioned above and use it to orchestrate a CSRF attack on a victim's session.

### Referer header weaknesses
>Referer headers are not mandatory, and some sites send requests without them. If the CSRF has no policy for handling headerless requests, attackers can use headerless requests to execute state-changing attacks.

This method has become less effective with the recent introduction of the referrer policy. This specification prevents URL leaks to other domains, by giving users more control over the information contained in the referrer header. They can choose to expose part of the referrer header information, or disable it by adding a metadata tag to the HTML page, as shown below:

```http
<meta name="referrer" content="no-referrer">
```
The above code suppresses the referrer header for all requests originating from this page. This makes it difficult for applications that rely on referrer headers to prevent CSRF attacks from such a page.

