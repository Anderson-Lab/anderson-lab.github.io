---
permalink: /csis_604_summer_2017/cluster_assignment/
title: "Distributed Computing"
excerpt: "Anderson Data Science Research Lab."
---

{% include base_path %}

## Creating the master VM

Before we can do anything in this course, we all need a distributed environment that is uniform. Now we could just require everyone to sign on to Amazon or Google Cloud and spin up a few machines, but this isn't about learning those technologies, which would just take a couple of seconds. It's about understanding what goes in to setting up a virtual environment. Another little technical consideration is whether to download the 32bit or 64bit ISO. If this were for a real application, we'd go with 64bit, but my VirtualBox install doesn't like 64bit VMs which is probably some hardware/bios related issue on my laptop. Not really important to figure out for our purposes.

To start you need to download the latest version of VirtualBox for your platform (). Then you need to download the Ubuntu server iso (). It's important to download the sever ISO as we will not be using the graphical interface. A very small number of you might have to reboot into the bios and enable virtualization technology (). Please please please install Ubuntu only in VirtualBox only unless you know what you are doing as I don't want you to mess with your machine. Here are the specs I used for my first virtual machine:

Name: master
CPU: 1
RAM: 1G
Hard Drive Dynamically sized 20G
Hostname: master

I think mounted the iso image in the CD drive under Storage. Once that is done, you are ready to install. I kept all of the defaults for the setup, and I just made a user called <i>lab</i> with a password of <i>password</i>. Basically, I just hit enter a bunch of times during the install. I did change the hostname from <i>ubuntu</i> to <i>master</i>. Other than that I kept the defaults. You do need to hit the arrow key to write the partition changes to the disk, so I couldn't just hit enter there. This occured twice during the install.

### Basic software that will be common to all machines
One of the first thinks we should configure is an ssh server. On your master VM login and type sudo apt-get install openssh-server. This will install a SSH server that runs on port 22. Here is a great link that you can follow to set up port forwarding (<a href="https://nsrc.org/workshops/2014/btnog/raw-attachment/wiki/Track2Agenda/ex-virtualbox-portforward-ssh.htm">link</a>. The client at the end is a program called Putty. A nice little program if you are a Windows user, but I prefer a program called MobaXterm, which is also free to download as the client. Up to you. If you are a Mac user, you already have ssh client in your terminal, so no need for Putty.

## Cloning
At this point, it is probably good to clone the master VM twice to create server1 and server2. This can be done by shutting down and right clicking on the VM. It's a good idea to check the reinitialize MAC Address option. Full clone is also probably good though it does take up more space. At this point, I think we should just leave the server1 and server2 VM off, and we'll finish with master.

### Setting up the network
Once your install is done, we need to completely shut down the VM in order to fix the master. Since this is the master and gateway machine for us, it needs two ethernet cards. One that is outward facing that will remain NAT, and the other that is internal. It is easy to add another ethernet card via the VirtualBox GUI. After you enable that and set it to internal network, we can start the master machine. At this point, you shouldn't be using VirtualBox to log into the machine. You should be using ssh. If you aren't then your life will be frustrating. Trust me. Get the ssh working :) At this point, here is what my networking looks like when I type sudo ifconfig:

<pre>lab@master:~$ sudo ifconfig
[sudo] password for lab:
enp0s3    Link encap:Ethernet  HWaddr 08:00:27:bd:fc:bd
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:febd:fcbd/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:122 errors:0 dropped:0 overruns:0 frame:0
          TX packets:98 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:14040 (14.0 KB)  TX bytes:17224 (17.2 KB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:160 errors:0 dropped:0 overruns:0 frame:0
          TX packets:160 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:11840 (11.8 KB)  TX bytes:11840 (11.8 KB)</pre>

enp0s3 is the first network adapter with NAT and it will always have IP 10.0.2.15. We want to configure the second network card, which is pretty easy. My second card is called enp0s8, which I found by typing sudo ifconfig -a. It's configuration needs to be added to /etc/network/interfaces, so use an editor that you like and open it (e.g., sudo nano /etc/network/interfaces or sudo vi /etc/network/interfaces). Here is what I added:
<pre>auto enp0s8
iface enp0s8 inet static
        address 192.168.0.1
        netmask 255.255.255.0
        broadcast 192.168.0.255</pre>

I then need to save this file and type sudo ifup enp0s8. If all is right it shouldn't give you any errors. Then we need to enable forwarding by editing /etc/sysctl.conf and uncomment:

# net.ipv4.ip_forward=1
so that it reads:

net.ipv4.ip_forward=1

save the file. And then run the following:
<pre>
sudo sysctl -p /etc/sysctl.conf

sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -m state --state RELATED,ESTABLISHED -j ACCEPT

sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT
</pre>
We really should be all set up, but we need to also save this so it doesn't get lost on reboot. Run the following:
<pre>
sudo sh -c "iptables-save > /etc/iptables.rules"

sudo sed -i 's/^exit 0/iptables-restore < \/etc\/iptables.rules\nexit 0/' /etc/rc.local
</pre>

At this point, you are ready to set up the server VMs.

## Server VM Setup
Now the server1 and server2 VM setup is pretty easy. First, we need to make sure the network is set to internal. Then we need to go in and assign a static ip address, gateway, and dns. While it is shutdown, make the change to the network via VirtualBox. Then boot each one up. Also, don't forget that master needs to be running all the time for things to work completely. My systems took a few minutes to boot up because it was trying to initialize the network.

### Change hostname
To change the hostname, edit /etc/hostname and then issue the command sudo hostname server1 or sudo hostname server2 depending on which machine you are setting up. 

Also, go into /etc/hosts and change 127.0.1.1 master to 127.0.1.1 server1. You should also add the following three lines to all of the VMs /etc/hosts:
<pre>192.168.0.1 master
192.168.0.100 server1
192.168.0.101 server2</pre>

### Set the network
For server1, change the interface options in /etc/network/interfaces to:
<pre>
auto enp0s3
iface enp0s3 inet static
        address 192.168.0.100
        netmask 255.255.255.0
        broadcast 192.168.0.255
        gateway 192.168.0.1
        dns-nameservers 8.8.8.8
</pre>

For server2, change the interface options in /etc/network/interfaces to:
<pre>
auto enp0s3
iface enp0s3 inet static
        address 192.168.0.101
        netmask 255.255.255.0
        broadcast 192.168.0.255
        gateway 192.168.0.1
        dns-nameservers 8.8.8.8
</pre>

## SSH Keys
At this point, it is really useful to add ssh keys so you can log in without passwords. On master:
<pre>
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
</pre>
Take all of the defaults and just keep hitting enter. Now set up server1 and server2.
<pre>
ssh-copy-id lab@server1

ssh-copy-id lab@server2

scp .ssh/id_rsa* server1:.ssh/

scp .ssh/id_rsa* server2:.ssh/
</pre>
Then ssh to server1 or server2 and issue:
<pre>
ssh-copy-id master
</pre>
At this point, you should be able to ssh to and from all the servers without passwords. You will need to copy the ssh keys to your local machine if you don't want to enter any passwords, but this is good enough I think. 

## What to turn in
You need to turn in evidence that all of the above is working. That means screenshots, etc for each part of this assignment. If there isn't enough detail, I will return without a grade. If there are any problems, please indicate them at the bottom of the report with the heading "Known Problems".
