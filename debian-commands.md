## Users & Groups

Create a group

~~~bash
sudo addgroup groupname
~~~

Add an Existing User to a Group

~~~bash
sudo usermod -a -G groupname username
~~~

How to Remove a User From a Group

~~~bash
sudo gpasswd -d username groupname
~~~


How to Delete a Group

~~~bash
sudo groupdel groupname
~~~


How to Change a User’s Primary Group

~~~bash
sudo usermod -g groupname username
~~~


## Manage file permissions


View file permissions

~~~bash
ls -lah
~~~

Change file permissions

Symbolic method
The first and probably easiest way is the relative (or symbolic) method, which lets you specify permissions with single letter abbreviations. A chmod command using this method consists of at least three parts from the following lists:

Access class |	Operator	| Access Type
u (user)	| + (add access) 	| r (read)
g (group)	| - (remove access)	| w (write)
o (other)	| = (set exact access)	| x (execute)
a (all: u, g, and o)

~~~bash
chmod a+r myfile
~~~




~~~bash

~~~




~~~bash

~~~




~~~bash

~~~




~~~bash

~~~





~~~bash

~~~





~~~bash

~~~




~~~bash

~~~




~~~bash

~~~




~~~bash

~~~




~~~bash

~~~




~~~bash

~~~
