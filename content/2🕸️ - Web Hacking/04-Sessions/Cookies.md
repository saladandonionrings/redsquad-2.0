<figure><img src="https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExNjU5ZWRlZjRkZTI3ZDczZWJjOTYwMzg5Y2M1M2EzNDc5YWU4NjMzMiZlcD12MV9pbnRlcm5hbF9naWZzX2dpZklkJmN0PWc/3o6MbitgftpbGFP3B6/giphy.gif" alt="" width="375"><figcaption></figcaption></figure>

## Testing for Cookies Attributes

Cookies are an essential part of web applications for maintaining session states and user preferences. However, if not configured properly, they can become a security risk. Here’s a guide on testing for cookies attributes and ensuring they are secure.

### Hacking Cookies

### Security Testing for Cookies

#### Restrict Access to Cookies

1. **Secure Attribute**
   - **Purpose:** Ensures the cookie is only sent to the server with an encrypted request over HTTPS. It is never sent with HTTP requests.
   - **Example:** 
```http
     Set-Cookie: name=value; Secure
```
   - **Testing:** Ensure the cookie is marked with the `Secure` attribute. Check if cookies are sent over HTTP by intercepting HTTP requests using a proxy tool like Burp Suite. If the cookie is sent over HTTP, it is not secure.

2. **HttpOnly Attribute**
   - **Purpose:** Prevents the cookie from being accessed through JavaScript via the `document.cookie` API, which helps mitigate XSS attacks.
   - **Example:** 
 ```http
     Set-Cookie: name=value; HttpOnly
```
   - **Testing:** Verify that cookies are marked with the `HttpOnly` attribute. Attempt to access cookies via JavaScript in the browser console:
 ```javascript
     console.log(document.cookie);
 ```
     If the cookie is not listed, it is protected.

#### Define Where Cookies Are Sent

3. **SameSite Attribute**
   - **Purpose:** Controls whether cookies are sent with cross-site requests, providing protection against CSRF attacks.
   - **Values:**
     - `Strict`: Cookies are only sent to the site from which they originated.
     - `Lax`: Cookies are sent when navigating from an external site, but not with third-party requests.
     - `None`: Cookies are sent with both first-party and third-party requests (requires `Secure` attribute if `None` is used).
   - **Example:**
 ```http
     Set-Cookie: name=value; SameSite=Strict
     Set-Cookie: name=value; SameSite=Lax
     Set-Cookie: name=value; SameSite=None; Secure
```
   - **Testing:** Check the `SameSite` attribute of cookies. Use browser developer tools to inspect cookies and ensure they are set correctly based on the intended security policy. Test cross-site requests and observe if the cookies are sent according to the `SameSite` policy.

#### Practical Example

Here's how you might configure a secure cookie in an HTTP response header:
```http
Set-Cookie: sessionid=abc123; Secure; HttpOnly; SameSite=Strict
```

#### Testing Tools
- **Browser Developer Tools:** Inspect cookies directly within the browser.
- **Burp Suite:** Intercept and analyze HTTP requests and responses.
- **OWASP ZAP:** Automated security testing tool with capabilities to inspect cookies.