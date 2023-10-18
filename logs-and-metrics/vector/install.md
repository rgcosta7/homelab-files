# Install Vector

## Installation script

~~~bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.vector.dev | bash
~~~


## APT for Debian, Ubuntu, and other Linux distributions


### Installation

First, add the Vector repo:

~~~bash
curl -1sLf \
  'https://repositories.timber.io/public/vector/cfg/setup/bash.deb.sh' \
| sudo -E bash
~~~

Then you can install the vector package:

~~~bash
sudo apt-get install vector
~~~


## Install Vector on Docker

Docker is an open platform for developing, shipping, and running applications and services. With Docker, you can manage your infrastructure in the same ways you manage your services. By taking advantage of Docker’s methodologies for shipping, testing, and deploying code quickly, you can significantly reduce the delay between writing code and running it in production. This page covers installing and managing Vector on the Docker platform.

### Installation

Pull the Vector image:

~~~bash
docker pull timberio/vector:0.33.0-debian
~~~

Vector is an end-to-end observability data pipeline designed to deploy under various roles. You mix and match these roles to create topologies. The intent is to make Vector as flexible as possible, allowing you to fluidly integrate Vector into your infrastructure over time. 


### Configure

Create a new Vector configuration. The below will output dummy logs to stdout.

~~~bash
cat <<-EOF > $PWD/vector.toml
[api]
enabled = true
address = "0.0.0.0:8686"

[sources.demo_logs]
type = "demo_logs"
interval = 1.0
format = "json"

[sinks.console]
inputs = ["demo_logs"]
target = "stdout"
type = "console"
encoding.codec = "json"
EOF
~~~

### Start

~~~bash
docker run \
  -d \
  -v $PWD/vector.yaml:/etc/vector/vector.yaml:ro \
  -p 8686:8686 \
  timberio/vector:0.33.0-debian
~~~


### Stop

~~~bash
docker stop timberio/vector
~~~

### Reload

~~~bash
docker kill --signal=HUP timberio/vector
~~~

### Restart

~~~bash
docker restart -f $(docker ps -aqf "name=vector")
~~~

### Observe

To tail the logs from your Vector image:

~~~bash
docker logs -f $(docker ps -aqf "name=vector")
~~~


