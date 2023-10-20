# Install Docker on Debian 10/11

Install using the Apt repository
Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker Apt repository. Afterward, you can install and update Docker from the repository.

### 1 - Set up Docker's Apt repository.

~~~bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
~~~

~~~bash
# Add the repository to Apt sources:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
~~~

Update repository

~~~bash
sudo apt-get update
~~~



## 2 - Install the Docker packages

To install the latest version, run:

~~~bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
~~~




## 3 - Verify that the installation is successful by running the hello-world image:

~~~bash
sudo docker run hello-world
~~~

This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.




## 4 - Configure Docker to start on boot with systemd

To automatically start Docker and containerd on boot for other Linux distributions using systemd, run the following commands:

~~~bash 
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
~~~



## 5 - Manage Docker as a non-root user

If you don't want to preface the docker command with sudo, create a Unix group called docker and add users to it. You will need to create the group if not created by the installation.

~~~bash 
sudo usermod -aG docker $USER
~~~



## 6 - Configure default logging driver

FFI:

Set the log driver to Promtail




## 7 - Create Docker network Macvlan

~~~bash
docker network create -d macvlan --subnet=10.0.20.0/27 --gateway=10.0.20.30 -o parent=ens18 media
~~~



## 8 - Install Local Persist Volume Plugin for Docker

Manual as root:

This is a manual installation and need to be run as a **ROOT**


 - 1 Download the binary files and give execute permission:

```bash
curl -fLsS "https://github.com/MatchbookLab/local-persist/releases/download/v1.3.0/local-persist-linux-amd64" > /usr/bin/docker-volume-local-persist
chmod +x /usr/bin/docker-volume-local-persist
```

- 2 Setup as a service: 

```bash
sudo curl -fLsS "https://raw.githubusercontent.com/MatchbookLab/local-persist/master/init/systemd.service" > /etc/systemd/system/docker-volume-local-persist.service
```

- 3 Enable and start the service

```bash
sudo systemctl enable docker-volume-local-persist
sudo systemctl start docker-volume-local-persist
sudo systemctl status --full --no-pager docker-volume-local-persist
```