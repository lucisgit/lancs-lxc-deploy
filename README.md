lancs-lxc-deploy
================

Script for automated deployment of LXC containers on Debian.

Prerequisites
-------------
You need to have some required packages installed on host:

    apt-get install lxc bridge-utils debootstrap

Next, you need to set up control groups:

    echo "none /cgroup cgroup defaults 0 0" >> /etc/fstab
    mkdir -p /cgroup
    mount /cgroup

Finally, set up network the way it is done in [sample interfaces file](https://github.com/lucisgit/lancs-lxc-deploy/blob/master/etc/network/interfaces).

Usage
-----
It is important to understand that LXC container deployment and creation are two different things. The given script does the deployment, i.e. it will create a configured debootstrap image with container configuration in the location specified in script parameter. To actually turn this into LXC container and use, you will need to run _lxc-create_ command that will produce a 'system object' for this container (by default located at _/var/lib/lxc/containername_ directory). If you need to modify container config file, (e.g. change name or IP) you need to do that at the location where container is deployed, after which destroy (using _lxc-destroy_) the container system object and create it again. If you need to access working LXC container file system directly from host, simply do that via deployed container rootfs directory (e.g. _/lxc/somecontiner/rootfs_).

The lancs-lxc-deploy is designed to be run by root. The script does two main tasks, one - it is creating debootstrap image of desired architecture, two - it configures image and creating cofiguration (config and fstab). Task one takes time, task two is fast. You may run script using existing container location, in which case script will only perform task two according to configuration given in script parameters. You might have guessed correctly, that once you have a deployed container, you may simply copy it to different location, then run the script that will reconfigure it to different name and IP.

Let us have a real example:

    root@mozart:/lxc# ./lancs-lxc-deploy apache-wheezy.local /lxc/apache-wheezy.local amd64 wheezy 192.168.254.15/24 br1
    Fetching filesystem...
    I: Retrieving Release
    I: Retrieving Release.gpg
    ...
    Removing udev ...
    Purging configuration files for udev ...
    Processing triggers for man-db ...
    Set root password for the container:
    Enter new UNIX password: 
    Retype new UNIX password: 
    passwd: password updated successfully
    We are done. Container template apache-wheezy.local has been created at /lxc/apache-wheezy.local. You may create and start your container now.
    Some useful tips:
    Create container:
        lxc-create -f /lxc/apache-wheezy.local/config -n apache-wheezy.local
    Destroy contaner:
        lxc-destroy -n apache-wheezy.local
    Start container on background:
        lxc-start -d -n apache-wheezy.local
    Get console acceess to the container (Ctrl+a q to leave):
        lxc-console -n apache-wheezy.local
    Stop container:
        lxc-stop -n apache-wheezy.local
    Stop contaier nicer way:
        lxc-halt -n apache-wheezy.local

Now container has been deployed. Note choosen IP address should belong to br1 IP range. You may create LXC system object:

    root@mozart:/lxc# lxc-create -f /lxc/apache-wheezy.local/config -n apache-wheezy.local
    'apache-wheezy.local' created

Start it on background:

    root@mozart:/lxc# lxc-start -d -n apache-wheezy.local

Access it (to quit press ctrl-a, then q):

    root@mozart:/lxc# lxc-console -n apache-wheezy.local

The safest way to stop container is to shut it down having logged in to it. There is information on the internet that even _lxc-halt_ designed for safe stop does not do it exactly right. No idea if this is correct or not.
    
Now, let us see how we can modify the deployed container. The easiest way if you need to change IP or hostname, would be to run _lancs-lxc-deploy_ on the same location again with different parameters:

    root@mozart:/lxc# ./lancs-lxc-deploy apache-wheezy.local /lxc/apache-wheezy.local amd64 wheezy 192.168.254.16/24 br1
    It seems that /lxc/apache-wheezy.local/rootfs already exists.
    What do you want to do?
      1. Delete /lxc/apache-wheezy.local/rootfs and create new container template.
      2. Update configuration of existing "apache-wheezy.local" container with the new settings.
      3. Quit.
    Enter your choice? 2
    Generating container main config file...
    Generating container fstab file...
    Setting hostname...
    Setting /etc/hosts...
    Creating device nodes...
    Setting locale...
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    locales is already the newest version.
    0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
    Installing packages...
    ...
    Set root password for the container:
    Enter new UNIX password: 
    Retype new UNIX password: 
    passwd: password updated successfully
    We are done. Container template apache-wheezy.local has been created at /lxc/apache-wheezy.local. You may create and start your container now.
    Some useful tips:
    Create container:
        lxc-create -f /lxc/apache-wheezy.local/config -n apache-wheezy.local
    Destroy contaner:
        lxc-destroy -n apache-wheezy.local
    Start container on background:
        lxc-start -d -n apache-wheezy.local
    Get console acceess to the container (Ctrl+a q to leave):
        lxc-console -n apache-wheezy.local
    Stop container:
        lxc-stop -n apache-wheezy.local
    Stop contaier nicer way:
        lxc-halt -n apache-wheezy.local

The configuration has changed, now you may destroy system object and create a new one (we stopped contaier before).

    root@mozart:/lxc# lxc-destroy -n apache-wheezy.local
    root@mozart:/lxc# lxc-create -f /lxc/apache-wheezy.local/config -n apache-wheezy.local

If you are cloning existing continer, simply copy it to different location and use that location in script paremeter anong with new name and IP. The rest of procedure will be the same as above, but you will not need to run destry as your container never existed.

The last bit I would like to point, that container configuration is pretty powerful. By default it is basic and pretty safe. For the full list of sessings see man page on lxc.conf. One useful setting is:

    lxc.mount.entry = /home/youruser /lxc/apache-wheezy.local/rootfs/home/youruser none defaults,bind 0 0
    
This makes your home dir mounted for accessing it from inside the container.

Useful resources
----------------
* http://lxc.sourceforge.net/man/lxc.html (or _man 7 lxc_) - must read!
* http://glonek.co.uk/tips-tricks/lxc-on-ubuntu-howto-tutorial/
* http://wiki.pcprobleemloos.nl/using_lxc_linux_containers_on_debian_squeeze/creating_a_lxc_virtual_machine_template
* https://wiki.debian.org/LXC
* http://lxc.teegra.net/#_container_install
* http://theangryangel.co.uk/blog/lxc-linux-containers/

