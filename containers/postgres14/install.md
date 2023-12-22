## Update Operating System

Update your Debian 12 operating system to make sure all existing packages are up to date:

~~~bash
sudo apt update && sudo apt upgrade -y
~~~


## Install PostgreSQL 14 using the official repository


PostgreSQL is available in the default Debian repositories but not always the available versions are up to date.

To get the latest packages of PostgreSQL, you must add the official repo of PostgreSQL.

First, install all required dependencies by running the following command:

~~~bash
sudo apt install wget sudo curl gnupg2 -y
~~~

After installing all the dependencies, create the file repository configuration with the following command:

~~~bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
~~~

Now import the repository signing key by using the following command:

~~~bash
sudo wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
~~~

Update your local package index with the following command:

~~~bash
sudo apt -y update
~~~

Now you can run the following command to install the latest version of the PostgreSQL server:

~~~bash
sudo apt install postgresql-14 -y
~~~

After the successful installation, start the PostgreSQL service and enable it to start after the system reboot:

~~~bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
~~~

Verify that is active and running on your server:

~~~bash
sudo systemctl status postgresql
~~~

~~~bash
The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Dec 20 19:09:45 UTC 2023 on tty1
sysadmin@postgres:~$ sudo systemctl status postgresql
[sudo] password for sysadmin: 
* postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; preset: enable>
     Active: active (exited) since Wed 2023-12-20 19:18:50 UTC; 2min 6s ago
    Process: 299 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 299 (code=exited, status=0/SUCCESS)
        CPU: 1ms
~~~



By default, PostgreSQL listens on port 5432. You can check it with the following command:

~~~bash
sudo ss -antpl | grep 5432
~~~

~~~
LISTEN 0      244        127.0.0.1:5432      0.0.0.0:*    users:(("postgres",pid=128,fd=6))
LISTEN 0      244            [::1]:5432         [::]:*    users:(("postgres",pid=128,fd=5))
~~~

## Enable remote access to Postgres

Open the PostgreSQL configuration file “postgresql.conf” using your preferred text editor. The file is typically located in the /etc/postgresql/15/main directory. To open the file from the Linux Terminal, execute:

~~~bash
sudo nano /etc/postgresql/14/main/postgresql.conf
~~~

Then, find the line #listen_addresses = 'localhost' and uncomment it (remove the # character at the beginning of the line).

Next, change the value of “listen_addresses” to “*”. This allows PostgreSQL to listen on all available IP addresses. Alternatively, you can specify a specific IP address or a range of IP addresses that are allowed to connect to the server.

~~~bash
listen_addresses = '*'                  # what IP address(es) to listen on;
~~~


Open the “pg_hba.conf” file using your preferred text editor. The file is typically located in the /etc/postgresql/15/main directory. To open the file from the Linux Terminal, execute: 

~~~bash
sudo nano /etc/postgresql/14/main/pg_hba.conf
~~~

And modify the follow line  # IPv4 local connections:

~~~bash
host    all             all             0.0.0.0/0            md5
~~~

Next, restart PostgreSQL by running the following command:

~~~bash
sudo service postgresql restart
~~~

## Configure SSL on PostgreSQL (My Keys)

For SSL to work with PostgreSQL you need to generate three certificate files:

    - server.key - This is the private key file
    - server.crt - This is the server certificate file
    - root.crt - This is the trusted root certificate

First, change the directory to PostgreSQL’s data directory as shown.

~~~bash
sudo mkdir /etc/postgresql/14/data
~~~

Upload you server keys to your home directory using WinSCP and copy them to the above directory. 

~~~bash
sudo cp * /etc/postgresql/14/data/
~~~

Next, for enhanced security, you need to assign read-only permissions of the private key to the root user as shown

~~~bash
sudo chmod 400 /etc/postgresql/14/data/*
~~~

In addition, set the ownership of the key to postgres user and group

~~~bash
sudo chown postgres:postgres /etc/postgresql/14/data/*
~~~

The next step is to configure PostgreSQL to use SSL. Access the postgresql.conf configuration file which is located inside the data directory.


~~~bash
sudo nano /etc/postgresql/14/main/postgresql.conf
~~~

In the SSL section, uncomment the following parameters and set the values as shown.

~~~bash
# - SSL -

ssl = on
ssl_ca_file = '/etc/postgresql/14/data/ca.crt'
ssl_cert_file = '/etc/postgresql/14/data/cert.pem'
#ssl_crl_file = ''
#ssl_crl_dir = ''
ssl_key_file = '/etc/postgresql/14/data/pub.key'
#ssl_ciphers = 'HIGH:MEDIUM:+3DES:!aNULL' # allowed SSL ciphers
~~~

Open the pg_hba.conf configuration file. This is the PostgreSQL client authentication configuration file that specifies which hosts are allowed to connect and how clients are authenticated.

~~~bash
sudo nano /etc/postgresql/14/main/pg_hba.conf
~~~

Under IPv4 local connections, add the following line at the end of the file to enable SSL and also allow connection from all hosts.

~~~bash
hostssl	 all         all          0.0.0.0/0    		md5
~~~

Save the changes and exit the configuration file. For the changes to come into effect, restart PostgreSQL.

~~~bash
sudo systemctl restart postgresql
~~~


## Logging into PostgreSQL

The Postgres user uses the ident authentication method and no password is set. For security reasons, it is recommended that you set a password to prevent potential breaches.

To do so, switch to the root user
~~~bash
sudo su
~~~

Next, switch to the postgres user.

~~~bash
su - postgres
~~~

Switch to the PostgreSQL shell


~~~bash
psql
~~~

Then set the postgres user’s password using the ALTER query as shown.

~~~bash
ALTER USER postgres WITH PASSWORD 'password';
~~~

