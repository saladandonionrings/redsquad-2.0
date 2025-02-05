# Overview
>Information disclosure, also known as information leakage, is when a website unintentionally reveals sensitive information to its users. Depending on the context, websites may leak all kinds of information to a potential attacker, including:
>- Data about other users, such as usernames or financial information
>- Sensitive commercial or business data
>- Technical details about the website and its infrastructure

## Examples
- Revealing the names of hidden directories, their structure, and their contents via a `robots.txt` file or directory listing
- Providing access to source code files via temporary backups
- Explicitly mentioning database table or column names in error messages
- Unnecessarily exposing highly sensitive information, such as credit card details
- Hard-coding API keys, IP addresses, database credentials, and so on in the source code
- Hinting at the existence or absence of resources, usernames, and so on via subtle differences in application behavior

**Common sources**
- Files for web crawlers
	- `/robots.txt`
	- `/sitemap.xml`
- Directory listings
- Developer comments
- Error messages
- Debugging data
- Source code disclosure via backup files
- Insecure configuration
	- Trace Method enabled
		- Check for the disclosure of internal authentication headers in the response
- Version control history
	- .git exposed