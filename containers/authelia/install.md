

# Install Authelia

This steps will install authelia in a debian machine, can be physical or virtual.

## Prepare the machine

First we need to add authelias repository to our apt sources list. For that we need curl , gnupg and apt-transport-https to be installed:

~~~bash
sudo apt install curl gnupg apt-transport-https -y
~~~

Now we download and add the authelia repository key with:

~~~bash
curl -s https://apt.authelia.com/organization/signing.asc | sudo apt-key add -
~~~

And we add the authelia repo link to our apt sources lists:

~~~bash
echo "deb https://apt.authelia.com/stable/debian/debian/ all main" | sudo tee /etc/apt/sources.list.d/authelia-stable-debian.list
~~~

But as the key gets stored in the legacy trusted.gpg keyring, we need to move it to /usr/share/keyrings:

~~~bash
sudo apt-key list
~~~

**Important: Find the entry that say "Authelia" in the entry after uid. In my case its the first entry.**

*From here, we can export the key. Important: you have to take the last two blocks in the pub code, and combine it to 8 digits without a space.*

~~~bash
sudo apt-key export C8E4D80D | sudo gpg --dearmour -o /usr/share/keyrings/authelia.gpg
~~~

The following message will likely appear:

~~~
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
~~~

Now we can update our apt source file for the repository in ~~~ /etc/apt/sources.list.d/authelia-stable-debian.list ~~~ and add a ~~~ signed-by ~~~~ tag:

~~~bash
sudo nano /etc/apt/sources.list.d/authelia-stable-debian.list
~~~

Make sure to DELETE all contents of this file and then only copy & paste this one line in there:

~~~
deb [arch=amd64 signed-by=/usr/share/keyrings/authelia.gpg] https://apt.authelia.com/stable/debian/debian/ all main
~~~

Save and exit
Now update the repositories with:

~~~bash
sudo apt update
~~~

This should now update without any error.

Now, finally, we are ready to install Authelia! Run:

~~~bash
sudo apt install authelia -y
~~~

## Configure Authelia

Before we can run Authelia, we need to create some folders and a fresh configuration file. To do so we run a couple of commands as superuser:

~~~bash
sudo su
~~~

No, as root, create the secret folders:

~~~bash
mkdir /etc/authelia/.secrets/
mkdir /etc/authelia/.users/
mkdir /etc/authelia/assets/
~~~

Next we need to generate our secrets and passwords for the Authelia config. In order to be secure and have long keys, we auto-generate 64 character long random keys and save them directly in a file in /etc/authelia/.secrets/. This way we don't need to remeber the passcodes and have them stored securly outside the config.

### Create Authelia secrets

As you might not need all of the following keys and secrets, because you are not using all services, you can skip them. Just run the ones you need:

AUTHELIA_JWT_SECRET_FILE

~~~bash
tr -cd '[:alnum:]' < /dev/urandom | fold -w "64" | head -n 1 | tr -d '\n' > /etc/authelia/.secrets/jwtsecret
~~~

AUTHELIA_SESSION_SECRET_FILE

~~~bash
tr -cd '[:alnum:]' < /dev/urandom | fold -w "64" | head -n 1 | tr -d '\n' > /etc/authelia/.secrets/session
~~~

STORAGE_ENCRYPTION_KEY

~~~bash
tr -cd '[:alnum:]' < /dev/urandom | fold -w "64" | head -n 1 | tr -d '\n' > /etc/authelia/.secrets/storage
~~~

STORAGE_MYSQL_PASSWORD

~~~bash
tr -dc 'A-Za-z0-9!#$%&*#' </dev/urandom | head -c 64 | tr -d '\n' > /etc/authelia/.secrets/mysql
~~~

Important: The mysql password command is different. As we need the mysql password later to setup our database, it is important it is matching the requirements of also having special characters in there. Once generated read out and note the password somewhere save to use it later in this guide with:

~~~bash
cat /etc/authelia/.secrets/mysql
~~~

If you already have a mysql server and want to use that, add its password in here with:

~~~bash
nano /etc/authelia/.secrets/mysql
~~~

AUTHELIA_NOTIFIER_SMTP_PASSWORD_FILE

~~~bash
tr -cd '[:alnum:]' < /dev/urandom | fold -w "64" | head -n 1 | tr -d '\n' > /etc/authelia/.secrets/smtp
~~~


Important: If you already have a smtp password, you need to write it in this file plaintext and replace any existing characters in there:

~~~bash
nano /etc/authelia/.secrets/smtp
~~~

Or if you don't have a smtp provider yet, you may want to read out the previously generated password to use it in your smtp setup. For that just use:

~~~bash
cat /etc/authelia/.secrets/smtp
~~~


AUTHELIA_IDENTITY_PROVIDERS_OIDC_HMAC_SECRET

~~~bash
tr -cd '[:alnum:]' < /dev/urandom | fold -w "64" | head -n 1 | tr -d '\n' > /etc/authelia/.secrets/oidcsecret
~~~

AUTHELIA_IDENTITY_PROVIDERS_OIDC_ISSUER_PRIVATE_KEY_FILE

This generates the public and private key if you are going to use an OpenID Provider like cloudflare:

~~~bash
openssl genrsa -out /etc/authelia/.secrets/oicd.pem 4096
~~~
~~~bash
openssl rsa -in /etc/authelia/.secrets/oicd.pem -outform PEM -pubout -out /etc/authelia/.secrets/oicd.pub.pem
~~~

AUTHELIA_SESSION_REDIS_PASSWORD_FILE (optional)

~~~bash
tr -cd '[:alnum:]' < /dev/urandom | fold -w "64" | head -n 1 | tr -d '\n' > /etc/authelia/.secrets/redis
~~~

Create the environment variable mapping list file:
Next we need a file that lists all of your previously generated keyfiles and maps them to the Authelia ENV vars. You can find a list of all Authelia ENV vars here, if you need more.
Create a new secrets file with nano like this:

~~~bash
nano /etc/authelia/secrets
~~~

And copy and paste the following list of mapped ENV vars into it. Please note, the unused ENV vars are deactivated with a # but for completion are part of the setup. Activate and deactivate whatever ENV vars you need:

~~~bash
### Authelia ENV vars mapping:
# basics:
AUTHELIA_JWT_SECRET_FILE=/etc/authelia/.secrets/jwtsecret
AUTHELIA_SESSION_SECRET_FILE=/etc/authelia/.secrets/session

# storage / mysql:
AUTHELIA_STORAGE_ENCRYPTION_KEY_FILE=/etc/authelia/.secrets/storage
AUTHELIA_STORAGE_MYSQL_PASSWORD_FILE=/etc/authelia/.secrets/mysql

# smtp notifications:
AUTHELIA_NOTIFIER_SMTP_PASSWORD_FILE=/etc/authelia/.secrets/smtp

# openid identity provider:
AUTHELIA_IDENTITY_PROVIDERS_OIDC_HMAC_SECRET_FILE=/etc/authelia/.secrets/oidcsecret
AUTHELIA_IDENTITY_PROVIDERS_OIDC_ISSUER_PRIVATE_KEY_FILE=/etc/authelia/.secrets/oicd.pem

# if you use LDAP for users
#AUTHELIA_AUTHENTICATION_BACKEND_LDAP_PASSWORD_FILE=/etc/authelia/.secrets/ldap

# if you user REDIS for session management (like kubernetes)
#AUTHELIA_SESSION_REDIS_PASSWORD_FILE=/etc/authelia/.secrets/redis

# if you user REDIS SENTINEL for high availability session management (like kubernetes)
#AUTHELIA_SESSION_REDIS_HIGH_AVAILABILITY_SENTINEL_PASSWORD_FILE=/etc/authelia/.secrets/redissentinel

# if you use DUO for push notification
#AUTHELIA_DUO_API_INTEGRATION_KEY_FILE=/etc/authelia/.secrets/duointkey
#AUTHELIA_DUO_API_SECRET_KEY_FILE=/etc/authelia/.secrets/duosecretkey

# if you use postgress
#AUTHELIA_STORAGE_POSTGRES_PASSWORD_FILE=/etc/authelia/.secrets/postgress
#AUTHELIA_STORAGE_POSTGRES_SSL_KEY_FILE=/etc/authelia/.secrets/postgresskey

# if you want to set extra TLS cert (default none) for using without a proxy like traefik
#AUTHELIA_SERVER_TLS_KEY_FILE=/etc/authelia/.secrets/tlskey.pem
~~~

Read more about Authelia secrets here: https://www.authelia.com/configuration/methods/secrets/

Now we need to set the right priviliges for the .secret folder and files with:

~~~bash
chmod 600 -R /etc/authelia/.secrets/
chmod 600 /etc/authelia/secrets
~~~

And exit the superuser mode with:

~~~bash
exit
~~~

### Modify the systemd unit (.service) to use the ENV vars

In order to make Authelia use and run with our previously created ENV vars list, we need to modify its startup systemd script. For that open it with:

~~~bash
sudo nano /etc/systemd/system/authelia.service
~~~

This will open the already existing file which we need to modify so it looks exactly like this:

~~~
[Unit]
Description=Authelia authentication and authorization server
After=multi-user.target

[Service]
Environment=AUTHELIA_SERVER_DISABLE_HEALTHCHECK=true
EnvironmentFile=/etc/authelia/secrets
ExecStart=/usr/bin/authelia --config /etc/authelia/configuration.yml
SyslogIdentifier=authelia

[Install]
WantedBy=multi-user.target
~~~

Save and exit, Now we just need to reload our serviced deamon to make the changes work:

~~~bash
sudo systemctl daemon-reload
~~~


## Create Authelia users

Next we create our login users. As in my case I am only using not more then three users, I will do the users in a very basic yml file based setup. If you want to do it via LDAP or AD check my optional LDAP configuration section here or check the official Authelia doc:

~~~bash
sudo nano /etc/authelia/.users/users_database.yml
~~~

Copy & paste this into that file:

~~~
###############################################################
#                         Users Database                      #
###############################################################

# This file can be used if you do not have an LDAP set up.

# List of users
users:
  admin:
    displayname: "Firstname Lastname"
### PASSWORD is an EXAMPLE and spells: "authelia"
    password: "$argon2id$v=19$m=65536,t=1,p=8$azlRYWI4RktCK0JQaWJVbg$fh+TDkULjzvGi6hb+5clAOGjJVjUbgxbm0/WiS+i308"
    email: user@example.com
    groups:
      - admins
      - dev
  bob: 
    displayname: "Bob Dylan" 
    password: "$argon2id$v=19$m=65536,t=3,p=2$BpLnfgDsc2WD8F2q$o/vzA4myCqZZ36bUGsDY//8mKUYNZZaR0t4MFFSs+iM" 
    email: bob.dylan@authelia.com 
    groups: 
      - dev 
  james: 
    displayname: "James Dean" 
    password: "$argon2id$v=19$m=65536,t=3,p=2$BpLnfgDsc2WD8F2q$o/vzA4myCqZZ36bUGsDY//8mKUYNZZaR0t4MFFSs+iM" 
    email: james.dean@authelia.com
~~~


You can gerneate your own hashed password with follow command:

~~~bash
authelia hash-password
~~~

This will return for example a hashed string like this:

~~~bash
$ authelia hash-password
Enter Password:
Confirm Password:

Digest: $argon2id$v=19$m=65536,t=3,p=4$4P1R3MoAcuRETTTO=5xTlA$jqzk81SYCRTB3v7+H9hB7LMerpOUUkG5z47lsUM1K3Y
~~~

Use that for your own password and copy and paste it in the same manor as in the above user config example.

**IMPORTANT: use a valid email address for your user, as this is the mail you will get reset links sent to.**

Save and exit

Lastly set the right priviliges for the .users folder and files with:


~~~bash
chmod 600 -R /etc/authelia/.users/
~~~

## Create your configuration.yml


First we create a backup of the default Authelia configuration.yml

~~~bash
sudo mv /etc/authelia/configuration.yml /etc/authelia/sample.configuration.yml
~~~

Next we open a blank fresh configuration.yml with:

~~~bash
sudo nano /etc/authelia/configuration.yml
~~~

And now you need to copy the configuration.yml, EDIT to your domain and paste to your config file.

Save and exit

Lastly we should also set secure access rights to our configuration file:

~~~bash
sudo chmod 600 /etc/authelia/configuration.yml
~~~


## Install mysql database


If you don't have a mariaDB installed follow here:  TO COME!!!


### Config mariaDB for authelia

Now we just need to create our working authelia database. Login to mysql again with our new mysql root password:

~~~bash
sudo mysql -u root -p
~~~

and create the new authelia database:

~~~bash
CREATE DATABASE authelia;
~~~

Next we are going to create a new mysql user to avoid using root user and grant the new user priviliges for the new database authelia. Here we use our previously generated and noted password! :

~~~bash
CREATE USER 'authy'@'localhost' IDENTIFIED BY 'Mynewpassword#13';
~~~

And grant the priviliges:

~~~bash
GRANT ALL PRIVILEGES ON * . * TO 'authy'@'localhost';
~~~

Now make the changes active by flushing priviliges:

~~~bash
FLUSH PRIVILEGES;
~~~


Great, lastly test the new user priviliges with:

~~~bash
SHOW GRANTS FOR 'authy'@'localhost';
~~~

And exit the mysql cli:

~~~bash
exit
~~~



## Startup Authelia

Finally, you made it! Its now time to start and run Authelia. Simply enter:

~~~bash
sudo systemctl restart authelia
~~~

In order to debug or if issues encounter, please use these two points to validate and check the error messages from Authelia:

1. The systemd status:

~~~bash
sudo systemctl status authelia
~~~

2. The Authelia debug log:

~~~bash
sudo nano /home/authy/authelia.log
~~~

## Optional: set your own Authelia assets


You can slightly modify the authelia look. So besides the configuration.yml option of theme you can copy two files to ~~~/etc/authelia/assets/~~~ that will be used instead of the default ones:

favicon.ico

logo.png


Copy your favorite icon and logo with the EXACT from above to this folder and authelia will use it after a 
restart:

~~~bash
sudo systemctl restart authelia
~~~
