## Using the PostgreSQL Backend

Create an new (empty) database for vaultwarden:

~~~sql
CREATE DATABASE vaultwarden;
~~~


Create an new (empty) database for vaultwarden:

~~~sql
CREATE USER vaultwarden WITH ENCRYPTED PASSWORD 'yourpassword';
GRANT ALL ON DATABASE vaultwarden TO vaultwarden;
GRANT all privileges ON database vaultwarden TO vaultwarden;
~~~

Configure vaultwarden and start it, so diesel can run migrations and set up the schema properly. Do not do anything else.