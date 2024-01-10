## MySQL commands

### Create a new User

~~~sql
CREATE USER USER_NAME@'IP' IDENTIFIED BY 'PASSWORD';
~~~

~~~sql
FLUSH PRIVILEGES;
~~~

% - from local

'*' - from any

IP - from IP


### Create Database

~~~sql
create database DATABASE_NAME;
FLUSH PRIVILEGES;
~~~


### Grant database to User

~~~sql
GRANT all privileges on DATABASE_NAME.* TO USER_NAME@'%' identified by 'PASSWORD';
FLUSH PRIVILEGES;
~~~


### Change user password

~~~sql
SET PASSWORD FOR 'MYUSER'@'IP-ADDRESS' = PASSWORD 'new_password'; 
FLUSH PRIVILEGES;
~~~

### Delete User

~~~sql
DROP USER MYUSER;
~~~