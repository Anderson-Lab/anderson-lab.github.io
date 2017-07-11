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
<pre>
export HADOOP_COMMON_HOME=/usr/local/hadoop
export HADOOP_MAPRED_HOME=$HADOOP_COMMON_HOME
export HADOOP_HDFS_HOME=$HADOOP_COMMON_HOME
export YARN_HOME=$HADOOP_COMMON_HOME
export PATH=$PATH:$HADOOP_COMMON_HOME/bin
export PATH=$PATH:$HADOOP_COMMON_HOME/sbin
</pre>
If you want, you could setup a soft link to a shared .bashrc located in /shared, but that is up to you.

### Configuration files
Now we are going to edit configuration files that are located in /usr/local/hadoop/etc/hadoop. 

To get the configuration below working I had to change /etc/hosts on master. Specifically, I had to change
<pre>
127.0.1.1 master
</pre>
to
<pre>
127.0.1.1 localhost
</pre>
I had to do the same thing on server1 and server2.

I one point I also had to remove a temporary directory on server1 and server2. Important thing is to read the log files and Google is your friend. Here is what I had to do: sudo rm -rf /tmp/hadoop-lab

#### core-site.xml
Set up the namenode. Add the following to the configuration section.
<pre>
  &lt;property&gt;
    &lt;name&gt;fs.defaultFS</name>
    &lt;value&gt;hdfs://master/&lt;/value&gt;
    &lt;description&gt;NameNode URI&lt;/description&gt;
  &lt;/property&gt;
</pre>

#### yarn-site.xml
Set up the YARN resource manager on master.
<pre>
&lt;property&gt;
  &lt;name&gt;yarn.resourcemanager.hostname</name>
  &lt;value&gt;master&lt;/value&gt;
  &lt;description&gt;The hostname of the ResourceManager&lt;/description&gt;
&lt;/property&gt;
&lt;property&gt;
  &lt;name&gt;yarn.nodemanager.aux-services</name>
  &lt;value&gt;mapreduce_shuffle&lt;/value&gt;
  &lt;description&gt;shuffle service for MapReduce&lt;/description&gt;
&lt;/property&gt;
</pre>

#### hdfs-site.xml
Descriptions are in the descriptions :)

<pre>
    &lt;property&gt;
        &lt;name&gt;dfs.datanode.data.dir</name>
        &lt;value&gt;file:///home/lab/hdfs/&lt;/value&gt;
        &lt;description&gt;DataNode directory for storing data chunks.&lt;/description&gt;
    &lt;/property&gt;

    &lt;property&gt;
        &lt;name&gt;dfs.namenode.name.dir</name>
        &lt;value&gt;file:///home/lab/hdfs/&lt;/value&gt;
        &lt;description&gt;NameNode directory for namespace and transaction logs storage.&lt;/description&gt;
    &lt;/property&gt;

    &lt;property&gt;
        &lt;name&gt;dfs.replication</name>
        &lt;value&gt;2&lt;/value&gt;
        &lt;description&gt;Number of replication for each chunk.&lt;/description&gt;
    &lt;/property&gt;
</pre>

#### mapred-site.xml
First, copy the template file over: cp mapred-site.xml.template mapred-site.xml, and then add the following:
<pre>
&lt;property&gt;
  &lt;name&gt;mapreduce.framework.name</name>
  &lt;value&gt;yarn&lt;/value&gt;
  &lt;description&gt;Execution framework.&lt;/description&gt;
&lt;/property&gt;
</pre>

#### slaves
Delete localhost from the file and add server1 and server2.

#### Sync all the directories
<pre>
cd /usr/local
for i in `cat hadoop/etc/hadoop/slaves`; do 
  echo $i; rsync -avxP --exclude=logs hadoop/ $i:/usr/local/hadoop/; 
done
</pre>

#### Format the filesystem
<pre>
hdfs namenode -format
</pre>

#### Start HDFS/Yarn
Before you start hadoop, you must install Java on each of the datanodes (server1 and server2): sudo apt-get install default-jdk

Then do:
<pre>
start-dfs.sh
start-yarn.sh
</pre>
And check for and deal with any errors.

### Checking status
<pre>
hdfs dfsadmin -report
</pre>
My output was:
<pre>
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
</pre>

Final test is to see if our grep example worked. Now that we have a true distributed system, we need to create the directories and copy the files into hdfs (for more information: https://dzone.com/articles/top-10-hadoop-shell-commands):
<pre>
hadoop fs -mkdir /input
hadoop fs -put ~/input/* /input
hadoop fs -ls /input
/usr/local/hadoop/bin/hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep /input ~/grep_example 'principal[.]*'
hadoop fs -get ~/grep_example/ ~
</pre>

## Other notes
It's probably a good idea to install firefox on master: sudo apt-get install firefox. That way it is easy to use the Hadoop internal links. To get this to work, make sure you have a X11 client installed (Xming for windows or the one that comes with MobaXterm with windows or XQuartz with mac).

## Post Questions
What does each of the following do?
1. NodeManager
2. YARN
3. DataNode
