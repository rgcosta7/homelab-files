## How to mount SMB shares via systemd daemon

We will save our configuration in the /etc/systemd/system directory, which is the recommended location for units created by the administrator. 

The initial file we will create will describe the mount point. This will be achieved using a mount unit. It is important to note that the filename must adhere to the naming scheme outlined in the unit documentation. 
Essentially, this involves replacing "/" with "-". For example, /mnt/net/smb/myshare will become mnt-net-smb-myshare.mount. The contents of the file are quite straightforward. 
The "What" field specifies the source path, the "Where" field indicates the target, and the "Type/Options" field stores the cifs options that will be utilized for mounting the share.

Navigate to system directory:

~~~bash
cd /etc/systemd/system
~~~

Create a mount file that describe your mount 

~~~bash
sudo nano mnt-destination.mount
~~~

And here is a example:

~~~bash
[Unit]
Description=My Super mount daemon

[Mount]
What=//ipv4_address/shared/folder
Where=/mnt/destination
Type=cifs
Options=rw,uid=1000,gid=1000,credentials=/root/.smbcredentials,nobrl
DirectoryMode=0755

[Install]
WantedBy=multi-user.target
~~~

You don't have any other tasks here. To reload the configuration, you can utilize systemctl daemon-reload and examine the mount. However, currently, it is just a standard mount and there is no automatic mounting taking place. 

To enable auto-mounting, we require an automount unit. It follows the same naming convention, except the file extension should be .automount. This file includes the [Automount] section, and it must have one mandatory entry, which is Where:.


~~~bash
sudo nano mnt-destination.automount
~~~

~~~bash
[Unit]
Description=My Super mount daemon

[Automount]
Where=/mnt/destination

[Install]
WantedBy=multi-user.target
~~~

This is the unit we want to enable and start automatically, so we need to perform the following steps:

To reload the configuration:

~~~bash
systemctl daemon-reload
~~~

To start the unit – so we can use it right away

~~~bash
systemctl start mnt-destination.automount 
~~~

To enable the auto-start of the unit

~~~bash
systemctl enable mnt-destination.automount 
~~~