# Promtail agent


Promtail is an agent which ships the contents of local logs to a private Grafana Loki instance or Grafana Cloud. It is usually deployed to every machine that runs applications which need to be monitored.

## Install using Docker

~~~bash
# modify tag to most recent version
docker pull grafana/promtail:2.9.2
~~~



## Install the binary

Now we will create the Promtail service that will act as the collector for Loki.

We can also get the Promtail binary from the same place as Loki.

To check the latest version of Promtail, visit the [Loki releases page.](https://github.com/grafana/loki/releases/). a\
Make sure you have ~~~ unzip ~~~ installed in your system!

~~~bash
cd /usr/local/bin
~~~


~~~bash
# modify tag to most recent version
curl -O -L "https://github.com/grafana/loki/releases/download/v2.9.2/promtail-linux-amd64.zip"
~~~

Unzip the archive

~~~bash
unzip "promtail-linux-amd64.zip"
~~~

And allow the execute permission on the Promtail binary

~~~bash
sudo chmod a+x "promtail-linux-amd64"
~~~

### Create the Promtail config

Now we will create the Promtail config file.

~~~bash
sudo nano config-promtail.yml
~~~

And add this script,

~~~bash
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/*log
~~~

### Configure Promtail as a Service

Now we will configure Promtail as a service so that we can keep it running in the background.

Create user specifically for the Promtail service


~~~bash
sudo useradd --system promtail
~~~

Create a file called promtail.service

~~~bash
sudo nano /etc/systemd/system/promtail.service
~~~

a\
And add this script,

~~~bash
[Unit]
Description=Promtail service
After=network.target

[Service]
Type=simple
User=promtail
ExecStart=/usr/local/bin/promtail-linux-amd64 -config.file /usr/local/bin/config-promtail.yml

[Install]
WantedBy=multi-user.target
~~~

Enable and start promtail service

~~~bash
sudo systemctl enable promtail 
sudo systemctl start promtail 
~~~

You can check if promtail  is enable and running by doing:

~~~bash
sudo systemctl status promtail
~~~

**Since this example uses Promtail to read system log files, the promtail user won't yet have permissions to read them.**


So add the user promtail to the adm group


~~~bash
sudo usermod -a -G adm promtail
~~~

Verify that the user is now in the adm group

~~~bash
id promtail
~~~


If you will need to restart the new Promtail service


~~~bash
sudo systemctl restart promtail
~~~



