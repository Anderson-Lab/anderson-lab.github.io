---
permalink: /csis_604_summer_2017/gluster_assignment/
title: "Distributed Computing"
excerpt: "Anderson Data Science Research Lab."
---

{% include base_path %}

In this assignment, we will install and configure the Gluster File System. In gluster there is no master node or management node. Every node can be a server and/or a client. As you've seen in the papers, there are alternatives (Ceph, etc), but after hours of errors that seem to be bugs in ceph-deploy according to the interwebs, I've decided to have you work with Gluster instead. If anyone can get Ceph to work and puts together a tutorial, feel free to submit it to the Dropbox for Gluster on OAKS, and I'll award bonus points. But a word of caution that this might be a deep rabbit hole or a quick fix. No way for me to tell at this point. Anyways... Let's get back to Gluster, which works great :)

## Adding extra disks to VMs
It is a really bad idea to use your root file system for a distributed file system. Your system will be unresponsive if it is filled up, then you're in for a bad time. We'll want to add another hard drive to each of the VMS. This is pretty easy in VirtualBox. Just bring down each machine and add a new disk using the UI. It's under Storage under Settings. I added mine under the SATA controller. It won't really matter, but it will probably change the name of the device in Ubuntu, so put it under SATA. 

At boot you should now have /dev/sdb in /dev. Best to check this first. If it isn't there, check you added the disk correctly. Assuming it is there, you need to partition the device. Ignoring tuning parameters, we can use <code>sudo mkfs.xfs /dev/sdb</code> on each node. This just create the partition table and formats. We still need to mount this new device. On each node execute <code>sudo mkdir /mnt/disk2</code>. That is going to be our mount point. From there you need to open /etc/fstab and add:
<pre>
/dev/sdb /mnt/disk2     xfs defaults 0  0
</pre>
Then type <code>mount -a</code> to make sure everything is mounted. Any errors need to be fixed before moving on of course.

## Make sure all clocks are synchronized
We are going to use NTP synchronization to make sure all of are VMs share the same clock. Definitely a good idea if things get too out of wack then bad things happen. Here are some commands to execute on each node:
<pre>
sudo apt-get install -y ntp ntpdate ntp-doc
sudo ntpdate 0.us.pool.ntp.org
sudo hwclock --systohc
sudo systemctl enable ntp
sudo systemctl start ntp
</pre>

### Questions to answer during write-up:
1. Describe how NTP works.
2. Find one alternative to NTP and comment on the differences.

## Installing Gluster and Creating Peer Network
Of course we start by installing Gluster on each node:
<pre>
sudo apt-get install glusterfs-server
</pre>
Now we need to create our peer list. From the master node:
<pre>
sudo gluster peer probe server1
sudo gluster peer probe server2
</pre>
Check your peer status:
<pre>
sudo gluster peer status
</pre>

## Replicated Distributed Volume
Gluster allows you to create a variety of different types of distributed file systems. Our first one is a simple replicated distributed system. Here is how to create a volume of this type:
<pre>
sudo gluster volume create gluster1 transport tcp replica 2 server1:/mnt/disk2/gluster1_brick1 server2:/mnt/disk2/gluster1_brick1
</pre>
Once you've created it, you can start it with:
<pre>
sudo gluster volume start gluster1
</pre>
Then you can mount it after you create a mountpoint:
<pre>
sudo mkdir /mnt/gluster1
sudo mount -t glusterfs master:/gluster1 /mnt/gluster1
</pre>
And you've got it! Check out what is now available to you by looking at the output of <code>df -h</code>.

## Distributed Volume
If you don't want any replication, then that is easy:
<pre>
sudo gluster volume create gluster2 transport tcp server1:/mnt/disk2/gluster2_brick1 server2:/mnt/disk2/gluster2_brick2
</pre>
Then we follow the same pattern as before, but this volume is called gluster2. Once you've created it, you can start it with:
<pre>
sudo gluster volume start gluster2
</pre>
Then you can mount it after you create a mountpoint:
<pre>
sudo mkdir /mnt/gluster2
sudo mount -t glusterfs master:/gluster2 /mnt/gluster2
</pre>
And you've got it! Check out what is now available to you by looking at the output of <code>df -h</code>.

## Striped Volume
If you want to use stripping, you can do that with:
<pre>
sudo gluster volume create gluster3 transport tcp stripe 2 server1:/mnt/disk2/gluster3_brick1 server2:/mnt/disk2/gluster3_brick2
</pre>
Then we follow the same pattern as before, but this volume is called gluster3. Once you've created it, you can start it with:
<pre>
sudo gluster volume start gluster3
</pre>
Then you can mount it after you create a mountpoint:
<pre>
sudo mkdir /mnt/gluster3
sudo mount -t glusterfs master:/gluster3 /mnt/gluster3
</pre>
And you've got it! Check out what is now available to you by looking at the output of <code>df -h</code>.

## Questions to answer during write-up:
1. Describe how NTP works.
2. Find one alternative to NTP and comment on the differences.
3. What is a replicated distributed volume? Put a file in /mnt/gluster1 and go find it. How many copies are there?
4. How is that different from a distributed volume? Put a file in /mnt/gluster2 and go find it in the bricks. How many copies are there?
5. How is this different than a striped volume? What happens when you put a file into /mnt/gluster3?
