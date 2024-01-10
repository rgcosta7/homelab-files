## Create database

To create a database named test, run the following command:

~~~bash
CREATE DATABASE test;
~~~

To list all databases, run the following command:

~~~bash
\l
~~~

To switch the database to test, run the following command:

~~~bash
\c test 
~~~

To create a table (e.g accounts), run the following command:

~~~bash
CREATE TABLE accounts (
	user_id serial PRIMARY KEY,
	username VARCHAR ( 50 ) UNIQUE NOT NULL,
	password VARCHAR ( 50 ) NOT NULL,
	email VARCHAR ( 255 ) UNIQUE NOT NULL,
	created_on TIMESTAMP NOT NULL,
        last_login TIMESTAMP 
);
~~~

To list all tables, run the following command:

~~~bash
\dt
~~~

To exit from the shell, run the following command:

~~~bash
exit
~~~

## Delete database

Once a database is no longer needed, you can drop it by using the DROP DATABASE statement.

The following illustrates the syntax of the DROP DATABASE statement:

~~~bash
DROP DATABASE [IF EXISTS] 'database_name';
~~~

Drop a database that has active connections example

~~~bash
SELECT *
FROM pg_stat_activity
WHERE datname = 'database_name';
~~~


## Backup and Restore a Single Database

You can back up and restore a single database using the pg_dump utility.

For example, to back up a single database named test and generate a backup file named test_backup.sql, run the following command:

~~~bash
su - postgres
pg_dump -d test -f test_backup.sql
~~~

You can also restore a single database using psql command.

For example, to restore a single database named test from a backup file named test_backup.sql, run the following command:

~~~bash
su - postgres
psql -d test -f test_backup.sql
~~~


## Create new User

CREATE USER adds a new user to a PostgreSQL database cluster.

Use ALTER USER to change the attributes of a user, and DROP USER to remove a user. Use ALTER GROUP to add the user to groups or remove the user from groups.

PostgreSQL includes a program createuser that has the same functionality as CREATE USER (in fact, it calls this command) but can be run from the command shell.

Create a user with no password:

~~~bash
CREATE USER jonathan;
~~~

Create a user with a password:

~~~bash
CREATE USER davide WITH PASSWORD 'jw8s0F4';
~~~

Create an account where the user can create databases:

~~~bash
CREATE USER manuel WITH PASSWORD 'jw8s0F4' CREATEDB;
~~~

Granting privileges on database to a user

~~~bash
GRANT all privileges on database mydb to myuser;
~~~

adding superuser permisson works

~~~bash
alter user <user> with superuser;
~~~