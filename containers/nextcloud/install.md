## Install nextCloud on Debian 11



~~~bash
sudo apt install unzip -y
~~~



### Install Apache webserver

Since Nexcloud will run on a web browser, the first step will be to install the Apache webserver. To achieve this, first update the system,

~~~bash
sudo apt update -y
~~~

Next, to install Apache execute the command

~~~bash
sudo apt install apache2 libapache2-mod-php -y
~~~


Once installed, verify the status of Apache  using the command

~~~bash
systemctl status apache2
~~~

### Install PHP

For Next Cloud to successfully run, we need to install PHP and other dependencies. To achieve this execute the command:

~~~bash
sudo apt-get install -y php php-gd php-curl php-zip php-dom php-xml php-simplexml php-mbstring php-mysql php-intl php-bz2 php-gmp php-imagick

~~~

To verify the version of PHP you have installed, run the command

~~~bash
php -v
~~~

As you can see in the output below, the latest PHP version (at the time of writing this article) is PHP 8.2


### Install MySQL database

A database will be required to accommodate the installation files of Next Cloud during the installation process. If you don't have a database runing already in your enviroment, follow the steps below to install the MySQL database.

~~~bash
apt install mysql-server php-mysql
~~~

Once the installation is complete, log in to MySQL as shown

~~~bash
mysql -u root -p
~~~

Next, create a database for Next Cloud as shown

~~~bash
CREATE DATABASE nextclouddb;

GRANT ALL ON nextclouddb.* TO 'nextcloud_user'@'localhost' IDENTIFIED BY 'strong password';

FLUSH PRIVILEGES;

EXIT;
~~~


### Download Nextcloud

With the database configured, now it’s time to download Nexcloud. First, navigate to the temporary folder

~~~bash
cd /tmp
~~~

Download Nextcloud zip file using the command

~~~bash
wget https://download.nextcloud.com/server/releases/latest.zip
~~~

Perfect!

Next, unzip the compressed file using the command

~~~bash
unzip latest.zip
~~~


Next, move the unzipped folder to the webroot directory as shown

~~~bash
sudo mv nextcloud/* /var/www/html
~~~

Navigate to the document root folder  and change the file ownership to www-data .

~~~bash
cd /var/www/html
sudo chown -R www-data:www-data *
~~~

Also, change the file permissions as shown

~~~bash
sudo chmod -R 755 *
~~~


### Install  Nextcloud via a browser

To install NextCloud on Debian 10 via a browser, open your favorite browser and browse your server’s IP as shown

http://iP_address/nextcloud/index.php