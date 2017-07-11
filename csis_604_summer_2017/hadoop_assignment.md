---
permalink: /csis_604_summer_2017/hadoop_assignment/
title: "Distributed Computing"
excerpt: "Anderson Data Science Research Lab."
---

{% include base_path %}

This week we are going to configure and implement some distributed applications in Hadoop. For this assignment, you'll need a basic level of command line expertise, and you'll need some Java Programming.

## Single Node Install
To install Hadoop and HDFS on master running as a single node, you can follow the link below. I will indicate where/if I deviated from the instructions.

<a href="https://www.digitalocean.com/community/tutorials/how-to-install-hadoop-in-stand-alone-mode-on-ubuntu-16-04">Single Node Instructions</a>

I didn't need to change anything, and if all works you should see hadoop spit out a ton of stuff when it runs, but no errors :) Be careful that the output directory doesn't exist. Now what did we run? Well. We ran the grep (search) example using Hadoop. Before that we transferred files from our local directory onto HDFS, and then we ran a map-reduce job. Later, we'll set up a development environment and see these things a little closer. 

## Cluster Setup
We are going to following the guidelines <a href="https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html">here</a>. Setting up the cluster hadoop is not as automated as the single node installation. The link above is a gernal overview of what is needed. We are setting up a simple system where we have two slaves (server1 and server2), and everything else can be configured on the master. In production, we'd follow the Hadoop guidelines a little closer, but performance is not our priority.

1. First, let's copy our Hadoop installation to our shared directory: sudo cp -Rp /usr/local/hadoop/ /shared
2. Now ssh to each server (server1 and server2), and copy it to /usr/local/. i.e., sudo cp -Rp /shared/hadoop /usr/local
3. In the .bashrc file on master, server1, or server2 add the following:
<code>
export HADOOP_COMMON_HOME=/usr/local/hadoop
export HADOOP_MAPRED_HOME=$HADOOP_COMMON_HOME
export HADOOP_HDFS_HOME=$HADOOP_COMMON_HOME
export YARN_HOME=$HADOOP_COMMON_HOME
export PATH=$PATH:$HADOOP_COMMON_HOME/bin
export PATH=$PATH:$HADOOP_COMMON_HOME/sbin
</code>
If you want, you could setup a soft link to a shared .bashrc located in /shared, but that is up to you.

### Configuration files
Now we are going to edit configuration files that are located in /usr/local/hadoop/etc/hadoop. 

To get the configuration below working I had to change /etc/hosts on master. Specifically, I had to change
<code>
127.0.1.1 master
</code>
to
<code>
127.0.1.1 localhost
</code>
I had to do the same thing on server1 and server2.

I one point I also had to remove a temporary directory on server1 and server2. Important thing is to read the log files and Google is your friend. Here is what I had to do: sudo rm -rf /tmp/hadoop-lab

#### core-site.xml
Set up the namenode. Add the following to the configuration section.
<code>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://master/</value>
    <description>NameNode URI</description>
  </property>
</code>

#### yarn-site.xml
Set up the YARN resource manager on master.
<code>
<property>
  <name>yarn.resourcemanager.hostname</name>
  <value>master</value>
  <description>The hostname of the ResourceManager</description>
</property>
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
  <description>shuffle service for MapReduce</description>
</property>
</code>

#### hdfs-site.xml
Descriptions are in the descriptions :)

<code>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///home/lab/hdfs/</value>
        <description>DataNode directory for storing data chunks.</description>
    </property>

    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///home/lab/hdfs/</value>
        <description>NameNode directory for namespace and transaction logs storage.</description>
    </property>

    <property>
        <name>dfs.replication</name>
        <value>2</value>
        <description>Number of replication for each chunk.</description>
    </property>
</code>

#### mapred-site.xml
First, copy the template file over: cp mapred-site.xml.template mapred-site.xml, and then add the following:
<code>
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
  <description>Execution framework.</description>
</property>
</code>

#### slaves
Delete localhost from the file and add server1 and server2.

#### Sync all the directories
<code>
cd /usr/local
for i in `cat hadoop/etc/hadoop/slaves`; do 
  echo $i; rsync -avxP --exclude=logs hadoop/ $i:/usr/local/hadoop/; 
done
</code>

#### Format the filesystem
<code>
hdfs namenode -format
</code>

#### Start HDFS/Yarn
Before you start hadoop, you must install Java on each of the datanodes (server1 and server2): sudo apt-get install default-jdk

Then do:
<code>
start-dfs.sh
start-yarn.sh
</code>
And check for and deal with any errors.

### Checking status
<code>
hdfs dfsadmin -report
</code>
My output was:
<code>
Configured Capacity: 38876733440 (36.21 GB)
Present Capacity: 32898555904 (30.64 GB)
DFS Remaining: 32898506752 (30.64 GB)
DFS Used: 49152 (48 KB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0

-------------------------------------------------
Live datanodes (2):

Name: 192.168.0.100:50010 (server1)
Hostname: server1
Decommission Status : Normal
Configured Capacity: 19438366720 (18.10 GB)
DFS Used: 24576 (24 KB)
Non DFS Used: 2989170688 (2.78 GB)
DFS Remaining: 16449171456 (15.32 GB)
DFS Used%: 0.00%
DFS Remaining%: 84.62%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon Jul 10 20:36:30 EDT 2017


Name: 192.168.0.101:50010 (server2)
Hostname: server2
Decommission Status : Normal
Configured Capacity: 19438366720 (18.10 GB)
DFS Used: 24576 (24 KB)
Non DFS Used: 2989006848 (2.78 GB)
DFS Remaining: 16449335296 (15.32 GB)
DFS Used%: 0.00%
DFS Remaining%: 84.62%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon Jul 10 20:36:31 EDT 2017
</code>

Final test is to see if our grep example worked. Now that we have a true distributed system, we need to create the directories and copy the files into hdfs (for more information: https://dzone.com/articles/top-10-hadoop-shell-commands):
<code>
hadoop fs -mkdir /input
hadoop fs -put ~/input/* /input
hadoop fs -ls /input
/usr/local/hadoop/bin/hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep /input ~/grep_example 'principal[.]*'
hadoop fs -get ~/grep_example/ ~
</code>

## Other notes
It's probably a good idea to install firefox on master: sudo apt-get install firefox. That way it is easy to use the Hadoop internal links. To get this to work, make sure you have a X11 client installed (Xming for windows or the one that comes with MobaXterm with windows or XQuartz with mac).
