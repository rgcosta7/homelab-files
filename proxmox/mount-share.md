## Mount a remote SMB shared folder

Download the package cifs:

```bash
sudo apt install cifs-utils -y
```

Store the user credencial on root:

```bash
sudo nano /root/.smbcredentials
```

~~~bash
username=username
password=password
~~~

Edit fstab to add mount point to auto mount in every boot:
Add the uid and gid of your user


```bash
sudo nano /etc/fstab
```

~~~bash
//ipv4_address/shared_folder /host_folder cifs uid=1000,gid=1000,credentials=/root/.smbcredentials,nobrl 0 0
~~~

```bash
sudo mkdir /host_folfer
```


```bash
sudo chown user_name:user_group /host_folder
```

Mount all: 

```bash
sudo mount -a -vvvv
```


## Mount a remote NFS shared folder