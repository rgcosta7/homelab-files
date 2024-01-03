## Proxmox USB Passthrough to an LXC 

The concept of USB pass through to an LXC container can be achieved by "mounting" the device within the container's environment. Nevertheless, this does not imply that the container can immediately interact with the device. It is also necessary to grant permission to the container for this purpose.

First we need to locate the device we want to pass through. lsusb could be used to find the connecting USB devices.

~~~bash
lsusb
~~~

Here is the output of my case:

~~~bash
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 013: ID 045e:0800 Microsoft Corp.
Bus 001 Device 020: ID 0634:5602 Micron Technology, Inc. CT4000X6SSD9
~~~

In this case, We going to passthrough the Crucial X6 4TB Portable SSD - CT4000X6SSD9

We are currently searching for the bus and device numbers, as they will help us locate the correct device path under /dev. Specifically, in this instance, the bus and device numbers are 001 and 020.

Consequently, the device path that needs to be mounted to the container should be /dev/bus/usb/001/020. To accomplish this, we can utilize the lxc.mount.entry option.

To enable the container to utilize the device path, it is essential to not only mount it but also grant permission through cgroup. Specifically, we can achieve this by utilizing the "lxc.cgroup.device.allow" option. To make use of this option, we must first determine the major and minor numbers of the device, which can be obtained in the following manner:

~~~bash
ls -al /dev/bus/usb/001/020
~~~

I got:

~~~bash
crw-rw-r-- 1 root root 189, 5 Dec 27 14:56 /dev/bus/usb/001/020
~~~

The major and minor numbers are 189 and 5 respectively.

### Start the configuration

In my case, I use Proxmox to manage my containers. I need to go to 

~~~bash
nano /etc/pve/lxc/<container_id>.conf .
~~~

I will have to put two lines more into the configuration file:

~~~bash
lxc.cgroup.devices.allow: c 189:* rwm
lxc.mount.entry: /dev/bus/usb/001/020 dev/bus/usb/001/020 none bind,optional,create=file
~~~

The former is for allowing the container privilege to access the device specified by its major and minor numbers.

Note: 189:* means we care only the major number, all the minors apply. rwm means read-write-mount.

The latter is for mounting the device file representation into the container space. In this case, I mount the exact device, however, it might be a good idea to map the whole directory (with all its siblings) to the container because it is more than likely that the filename will change.

Given that it can be done thus:

~~~bash
lxc.mount.entry: /dev/bus/usb/001 dev/bus/usb/001 none bind,optional,create=dir
~~~
