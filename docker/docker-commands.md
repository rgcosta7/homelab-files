# Docker commands good to know

## 1 - Access the Running Container’s Shell:

~~~bash
docker exec -it CONTAINER_ID /bin/bash
~~~


## 2 - Run a Command from Outside the Container


### Update the container:

~~~bash
docker exec CONTAINER_ID apt-get update && apt-get upgrade
~~~


### Check external IP address:

~~~bash
docker exec CONTAINER_ID curl ifconfig.co/
~~~


## 3 - Copy file from host to container:

~~~bash
docker cp index.html CONTAINER_ID:/usr/share/nginx/html/
~~~

## 4 - Create Docker network Macvlan

~~~bash
docker network create -d macvlan --subnet=10.0.20.0/27 --gateway=10.0.20.30 -o parent=ens18 media
~~~

## 3 

~~~bash

~~~





~~~bash

~~~



~~~bash

~~~



~~~bash

~~~