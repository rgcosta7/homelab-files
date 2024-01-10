## How to install a Minecraft Bedrock Edition in a LXC

### Setup

CREATE A CONTAINER WITH 2 CPU CORE AND 2GB MEMORY RAM. AS HDD I START WITH 6GB

### Intalation

Update and Upgrade the container:

~~~bash
sudo apt update && sudo apt upgrade -y
~~~

Install necessary files:
~~~bash
sudo apt install wget unzip screen openssl -y
~~~

Crate the Minecraft folder:

~~~bash
sudo mkdir /etc/craft
~~~

Grab the latest Bedrock version:

~~~bash
DOWNLOAD_URL=$(curl -H "Accept-Encoding: identity" -H "Accept-Language: en" -s -L -A "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; BEDROCK-UPDATER)" https://minecraft.net/en-us/download/server/bedrock/ |  grep -o 'https://minecraft.azureedge.net/bin-linux/[^"]*')
~~~

Download the latest version:

~~~bash
wget $DOWNLOAD_URL -O ~/bedrock-server.zip
~~~

Unzip the downloaded file to the directory:

~~~bash
sudo unzip ~/bedrock-server.zip -d /etc/craft
~~~

Give the folder the right permitions:

~~~bash
sudo chown <username>:<username> -R /etc/craft
~~~

Create the service file (easy to mantain):

~~~bash
sudo nano /etc/systemd/system/bedrock.service
~~~

~~~bash
[Unit]
Description=Minecraft Bedrock Service
After=network-online.target

[Service]
User=sysadmin
WorkingDirectory=/etc/craft
Type=forking
ExecStart=/usr/bin/screen -dmS bedrock /bin/bash -c "LD_LIBRARY_PATH=. ./bedrock_server"

ExecStop=/usr/bin/screen -Rd bedrock -X stuff "say is stopping in 10 seconds...\r"
ExecStop=/bin/sleep 1
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "say is stopping in 9 seconds...\r"
ExecStop=/bin/sleep 1
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "say is stopping in 8 seconds...\r"
ExecStop=/bin/sleep 1
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "say is stopping in 7 seconds...\r"
ExecStop=/bin/sleep 1
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "say is stopping in 6 seconds...\r"
ExecStop=/bin/sleep 1
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "say is stopping in 5 seconds...\r"
ExecStop=/bin/sleep 1
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "say is stopping in 4 seconds...\r"
ExecStop=/bin/sleep 1
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "say is stopping in 3 seconds...\r"
ExecStop=/bin/sleep 1
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "say is stopping in 2 seconds...\r"
ExecStop=/bin/sleep 1
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "say is stopping in 1 seconds...\r"
ExecStop=/bin/sleep 1
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "say Bye...\r"
ExecStop=/bin/sleep 1
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "save-all^M"
ExecStop=/usr/bin/screen -Rd bedrock -X stuff "stop^M"
ExecStop=/bin/sleep 5
GuessMainPID=no
TimeoutStartSec=30
TimeoutStopSec=30
Restart=on-failure

[Install]
WantedBy=multi-user.target
~~~

Edit Bedrock server  properties:

~~~bash
nano /etc/craft/server.properties
~~~


Enable the service:

~~~bash
sudo systemctl enable bedrock.service
~~~

Reload the systemd:

~~~bash
sudo systemctl daemon-reload
~~~

Start the Bedrock server:

~~~bash
sudo systemctl start bedrock.service
~~~

..and you have a Minecraft Bedrock version server running in a container.

## Upgrade the Server

sudo cp /etc/craft/server.properties /etc/craft/server.properties.bak
sudo cp /etc/craft/allowlist.json /etc/craft/allowlist.json.bak
sudo cp /etc/craft/permissions.json /etc/craft/permissions.json.bak

DOWNLOAD_URL=$(curl -H "Accept-Encoding: identity" -H "Accept-Language: en" -s -L -A "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; BEDROCK-UPDATER)" https://minecraft.net/en-us/download/server/bedrock/ |  grep -o 'https://minecraft.azureedge.net/bin-linux/[^"]*')

wget $DOWNLOAD_URL -O ~/bedrock-server.zip

sudo unzip -o ~/bedrock-server.zip -d /etc/craft

sudo mv /etc/craft/server.properties.bak /etc/craft/server.properties
sudo mv /etc/craft/allowlist.json.bak /etc/craft/allowlist.json
sudo mv /etc/craft/permissions.json.bak /etc/craft/permissions.json

sudo chown sysadmin /etc/craft -R


sudo systemctl start bedrock.service
