# Windows
- SharpHound.exe

# Linux
- Install : 
	- `bloodhound-python` (for data ingestion)
	- `BloodHound` (to visualize data)
	- `neo4j` ("database" like)

```bash
# install
# bloodhound-python
pip install bloodhound
# bloodhound
wget https://github.com/BloodHoundAD/BloodHound/releases/download/v4.3.1/BloodHound-linux-x64.zip
unzip BloodHound-linux-x64
# neo4j
wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/neotechnology.gpg
echo 'deb [signed-by=/etc/apt/keyrings/neotechnology.gpg] https://debian.neo4j.com stable latest' | sudo tee -a /etc/apt/sources.list.d/neo4j.list
sudo apt-get update
sudo apt-get install neo4j=1:5.25.1

# usages
bloodhound-python -c ALL -u USER -p PASSWORD -ns DCIP -d DOMAIN
neo4j console --verbose
# go to repo of BloodHound-linux-x64
./BloodHound
```