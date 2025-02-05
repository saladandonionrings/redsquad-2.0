# Basics

**Oracle APEX** is a low-code development platform used for building and deploying web applications. It integrates seamlessly with the Oracle Database, providing tools to quickly create data-driven applications using **PL/SQL**, along with other web technologies such as **HTML**, **CSS**, and **JavaScript**.

## Security Overview
Oracle APEX offers built-in security features to protect against common web vulnerabilities like SQL Injection and Cross-Site Scripting (XSS). However, the security of applications built with APEX depends heavily on:
- Proper input validation.
- Access control measures.
- Regular updates and patches.

# APEX URL Syntax

APEX URLs follow a specific syntax to define the application, page, session, and other parameters:
```
Application ID:Page ID:Session ID:Request:Debug:Clear Cache:Item Names:Item Values:Printer Friendly
```

**Example APEX URL:**
- Refers to **Page 1** of **Application 100**:
```text
  http://localhost/apex/f?p=100:1:12432087235079
```

# Interesting Endpoints

- **Admin Interface:**
  - `http://hostname:port/apex/apex_admin`
  - `http://hostname:port/pls/apex/apex_admin`
- **Application Builder:**
  - `http://hostname/ords/<workspace_name>/builder`

# Information Leak

APEX applications may expose certain details in source code or HTTP responses, such as:
- `APEX_VERSION`
- `application-version`
- `apex-version`

# Testing for Insecure Direct Object References (IDOR)

Insecure Direct Object References (IDOR) vulnerabilities allow unauthorized access to sensitive resources by manipulating identifiers in the URL.

## Identifying Application ID and Page ID
An example URL format is:
```text
https://my.app.com/apex/f?p=x:y:SESSION:::::ITEM:ITEM_VALUE
```
- **x** = Application ID
- **y** = Page ID

## Using Burp Suite for IDOR Testing

1. **Capture a request** with Burp Proxy and **send it to Intruder**.
2. Set your **payload position** on the **pageID** parameter.
3. Under **Payloads**, choose the "Numbers" payload and set an appropriate range for testing.
4. **Run** the attack and observe for unauthorized access.

# Testing for SQL Injection (SQLi)

Oracle APEX may be vulnerable to SQL Injection. To test for such vulnerabilities, tools like `sqlmap` can be used.

![[Pasted image 20240906113857.png]]

## Using `sqlmap` for SQLi Testing

- **APEX-specific SQL Injection:**
  To exploit vulnerabilities, you can rewrite the URL using `wwv_flow.show`:

```bash
sqlmap -u "https://app.oracle.com/ords/wwv_flow.show?p_flow_id=112&p_flow_step_id=5&p_instance=14720048029141&p_arg_name=RP,45&p_arg_value=F_DISPLAY" --batch --dbms Oracle --level 3 --risk 3
```

- **Explanation:**
  - `-u`: Specifies the target URL.
  - `--batch`: Non-interactive mode; assumes "yes" to all questions.
  - `--dbms Oracle`: Specifies the target DBMS is Oracle.
  - `--level 3`: Sets the testing intensity level.
  - `--risk 3`: Sets the risk level for payloads.

# Additional Resources
- **Testing for IDOR and Authorization Vulnerabilities in Oracle APEX:**  
  [Gray Tier Blog](https://graytier.com/blog/f/testing-for-idor-and-authorization-vulnerabilities-in-oracle-apex)

- **Hacking Oracle APEX Presentation (2018):**  
  [Presentation by Spendolini](https://www.neooug.org/gloc/Presentations/2018/SpendoliniHacking%20Oracle%20APEX.pdf)