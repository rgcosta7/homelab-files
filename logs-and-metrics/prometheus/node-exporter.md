
# Prometheus Node Exporter Setup

The Node Exporter is an agent that gathers system metrics and exposes them in a format which can be ingested by Prometheus. The Node Exporter is a project that is maintained through the Prometheus project. This is a completely optional step and can be skipped if you do not wish to gather system metrics. The following will need to be performed on each server that you wish to monitor system metrics for.

## Download Node Exporter


Download the Node Exporter binary to each Couchbase Server that you want to monitor. The Node Exporter will export system related stats.

~~~bash
# Visit the Prometheus downloads page for the latest version.

wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
~~~

## Create User

Create a Node Exporter user, required directories, and make prometheus user as the owner of those directories.

~~~bash
sudo groupadd -f node_exporter
sudo useradd -g node_exporter --no-create-home --shell /bin/false node_exporter
sudo mkdir /etc/node_exporter
sudo chown node_exporter:node_exporter /etc/node_exporter
~~~

## Extra

Prometheus supports basic authentication and TLS encryption (hhtps)

### SECURING PROMETHEUS USING BASIC AUTH

Let's say that you want to require a username and password from all users accessing the Prometheus instance. For this example, use admin as the username and choose any password you'd like.

First, generate a bcrypt hash of the password. To generate a hashed password, we will use python3-bcrypt.

Let's install it by running apt install python3-bcrypt, assuming you are running a debian-like distribution. Other alternatives exist to generate hashed passwords; for testing you can also use bcrypt generators on the web.

Here is a python script which uses python3-bcrypt to prompt for a password and hash it:

~~~bash
import getpass
import bcrypt

password = getpass.getpass("password: ")
hashed_password = bcrypt.hashpw(password.encode("utf-8"), bcrypt.gensalt())
print(hashed_password.decode())
~~~

Save that script as gen-pass.py and run it:

~~~bash
python3 gen-pass.py
~~~

### TLS ENCRYPTION

Let's also say that you've generated the following using OpenSSL or an analogous tool:

an SSL certificate at /etc/node_exporter/node_exporter.crt
an SSL key at /etc/node_exporter/node_exporter.key

Below is an example web-config.yml configuration file. With this configuration, Prometheus will serve all its endpoints behind TLS.

~~~bash
nano /etc/node_exporter/web.yml
~~~

~~~bash
# TLS and basic authentication configuration.
#
# Additionally, a certificate and a key file are needed.
tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key

# Usernames and passwords required to connect to Prometheus.
# Passwords are hashed with bcrypt: https://github.com/prometheus/exporter-toolkit/blob/master/docs/web-configuration.md#about-bcrypt
basic_auth_users:
  alice: $2y$10$NxXUUyeeeffefdOg32wQF7a4D61RCDnUv8saVOC3.xUKLzqEoS
  bob: $2y$10$hLqFl9jSjoAeey95Z/efef8Ye8wwekdMBM8c5Bn1ptYqP/AXyV0.oy0S8m
~~~

**Make sure node_exporter.crt and node_exporter.key are in the same folder as the web.yml**


## Unpack Node Exporter Binary

Untar and move the downloaded Node Exporter binary


~~~bash
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
mv node_exporter-1.6.1.linux-amd64 node_exporter-files
~~~

# Install Node Exporter

Copy node_exporter binary from node_exporter-files folder to /usr/bin and change the ownership to prometheus user.

~~~bash
sudo cp node_exporter-files/node_exporter /usr/bin/
sudo chown node_exporter:node_exporter /usr/bin/node_exporter
~~~

## Setup Node Exporter Service

Create a node_exporter service file.

~~~bash
sudo nano /usr/lib/systemd/system/node_exporter.service
~~~


Add the following configuration


~~~bash
[Unit]
Description=Node Exporter
Documentation=https://prometheus.io/docs/guides/node-exporter/
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
ExecStart=/usr/bin/node_exporter --web.config.file=/etc/node_exporter/web.yml

[Install]
WantedBy=multi-user.target
~~~


~~~bash
sudo chmod 664 /usr/lib/systemd/system/node_exporter.service
~~~

## Reload systemd and Start Node Exporter

Reload the systemd service to register the prometheus service and start the prometheus service.

~~~bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
~~~

Check the node exporter service status using the following command.


~~~bash
sudo systemctl status node_exporter
~~~


Configure node_exporter to start at boot

~~~bash
sudo systemctl enable node_exporter.service
~~~

If firewalld is enabled and running, add a rule for port 9100


## Verify Node Exporter is Running

~~~bash
https://<node_exporter-ip>:9100/metrics
~~~


## Clean Up


Remove the download and temporary files

~~~bash
rm -rf node_exporter-1.0.1.linux-amd64.tar.gz node_exporter-files
~~~