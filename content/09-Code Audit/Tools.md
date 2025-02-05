# SONARQUBE

>_Open-source quality code assessment tool (SAST)_

## Installation | Linux
### Install Sonarqube VM

https://bitnami.com/stack/sonarqube/virtual-machine

### Sonarqube-Scanner Installation
https://techexpert.tips/sonarqube/sonarqube-scanner-installation-ubuntu-linux

```bash
# download the sonarqube scanner and move it to /opt
apt-get update
apt-get install unzip wget nodejs

mkdir /downloads/sonarqube -p
cd /downloads/sonarqube
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.2.0.1873-linux.zip
unzip sonar-scanner-cli-4.2.0.1873-linux.zip
mv sonar-scanner-4.2.0.1873-linux /opt/sonar-scanner

# edit sonar-scanner.properties file
nano /opt/sonar-scanner/conf/sonar-scanner.properties

# add lines
sonar.host.url=http://localhost:9000
sonar.sourceEncoding=UTF-8

# create file
nano /etc/profile.d/sonar-scanner.sh

# add lines
#/bin/bash
export PATH="$PATH:/opt/sonar-scanner/bin"

# reboot
reboot
source /etc/profile.d/sonar-scanner.sh

# verify path
env | grep PATH
# output like
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/opt/sonar-scanner/bin

# verify version
sonar-scanner -v
```

## Create Project | Web

Go to `http://$vm_sonarqube_ip:9000`

_**Projects > Create Project (Manually) > (choose name and key) > locally > generate token > choose code language > OS > execute commands**_

## Docker

```bash
docker pull sonarqube:8.9-community
docker run -d — name sonarqube -p 9000:9000 sonarqube:8.9-community
docker ps -a
# go to http://localhost:9000
# admin:sonarqube
```

# BANDIT

>_Find common security issues in Python code._
>https://github.com/PyCQA/bandit

