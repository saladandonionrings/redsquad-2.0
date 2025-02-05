# Firefox

## Firefox Decrypt

> Extract passwords from Mozilla

https://github.com/unode/firefox_decrypt

### Install and Usage

```bash
git clone https://github.com/unode/firefox_decrypt
cd firefox_decrypt/
```

```bash
python firefox_decrypt.py [PATH = Path of the firefox extracted profile]
```

![[Pasted image 20240906171604.png]]

```bash
# Advanced usage

python firefox_decrypt.py --list
python firefox_decrypt.py /folder/containing/profiles.ini/
python firefox_decrypt.py | grep -C2 keyword
```

## firepwd

>Decrypt Mozilla protected passwords

https://github.com/lclevy/firepwd

```bash
# install 
git clone https://github.com/lclevy/firepwd.git
cd firepwd
pip3 install -r requirements.txt

# must have key4.db or key3.db & logins.json
# usage
python3 firepwd.py
```
