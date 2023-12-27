## Install MariaDB

Create a container and unselect ```Unprivileged container``` 
And the other options as normal, don't forget to set dns server.

Before start container, go to the container Options.

Login as root with password you setup and update your system:

```shell
sudo apt update && sudo apt upgrade -y
```

Install necessary packages:

```shell
sudo apt install sudo curl ca-certificates gnupg2 -y
```

Add non-Root user to the container:

```shell
sudo adduser username --ingroup users
```
Add user to the sudo list:

```shell
sudo usermod -a -G sudo username
```

Reboot and login with the new user

### Installing MariaDB server

```shell
curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
```

```shell
sudo apt install mariadb-server -y
```

### Change default setting on MariaDB

Edit 50-server.cnf

```shell 
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf 
```


### Secure MariaDB server

Run the secure script to set root password, remove test database and disable remote root user login.

```shell 
sudo mariadb-secure-installation 
```

