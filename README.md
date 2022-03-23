# Hadoop3 Multinode Cluster Installation on Ubuntu20.04

This is a guide for Hadoop3 3 Node Cluster installation on Ubuntu 20.04

## Step 1 - Install Java

If java is not installed on your servers, you can install java by following the instructions in;&#x20;

NOTE: We will be using java 8 for this project.&#x20;

{% embed url="https://github.com/ozgunakin/java-installation-on-ubuntu20.04" %}

## Step 2 - Configure Host Files

You need to configure host names and host files for each node.

* [x] Open the hosts file

```
sudo nano /etc/hosts
```

* [x] Add IP addresses and hostnames of each node in the cluster.

```
160.75.61.200 master
160.75.61.201 slave1
160.75.61.202 slave2
```

## Step 3 - Configure SSH

You need to install open ssh on each node and you need to configure a passwordless ssh connection between nodes.

* [x] Install Open SSH server and client.

```
sudo apt-get install openssh-server openssh-client
```

* [x] Generate Key Pairs on Master

```
ssh-keygen
```

* [x] Copy the content of .ssh/id_rsa.pub(of master) to .ssh/authorized\__keys (of all the slaves as well as master)&#x20;

```
ssh-copy-id user@master
ssh-copy-id user@slave1
ssh-copy-id user@slave2
```

* [x] Test your connection

```
ssh user@slave1
```

## Step 3 - Install Hadoop3

* [x] Download Hadoop

```
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz
```

* [x] Extract the file

```
tar -xvzf hadoop-3.3.1.tar.gz
```

* [x] Move the extracted file to the opt directory

```
sudo mv hadoop-3.3.1.tar.gz /opt/
```

* [x] Open .bashrc file

```
nano ~/.bashrc
```

* [x] Add following lines into .bashrc file

```
#HADOOP ENVIRONMENT VARIABLES
export HADOOP_HOME=/opt/hadoop-3.3.1
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export YARN_HOME=${HADOOP_HOME}
```

* [x] Source .bashrc file to apply the changes&#x20;

```
source ~/.bashrc
```

## Step 4 - Configure Hadoop3

We will configure 5 important files of Hadoop **on ALL NODES**.

#### File 1 : hadoop-env.sh

* [x] Open hadoop-env.sh file.

```
nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

* [x] Add following lines into hadoop-env.sh file

```
#JAVA HOME PATH
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

* [x] Create necessary directories for hadoop and change the owner of the directories.

```
sudo mkdir -p /usr/local/hadoop/hdfs/data
sudo chown ubuntu:ubuntu -R /usr/local/hadoop/hdfs/data
chmod 700 /usr/local/hadoop/hdfs/data
```

#### File 2 : core-site.xml

* [x] Open core-site.xml file.

```
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

* [x] Add following lines into core-site.xml file (Change MASTER-IP with the IP of your master node)

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://MASTER-IP:9000</value>
    </property>
</configuration>
```

#### File 3 : hdfs-site.xml

* [x] Open hdfs-site.xml file.

```
nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

* [x] Add following lines into hdfs-site.xml file

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///usr/local/hadoop/hdfs/data</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///usr/local/hadoop/hdfs/data</value>
    </property>
</configuration>
```

#### File 4 : mapred-site.xml

* [x] Open mapred-site.xml file.

```
nano $HADOOP_HOME/etc/hadoop/mapred-site.xml
```

* [x] Add following lines into hdfs-site.xml file (Change MASTER-IP with the IP of your master node)

```
<configuration>
    <property>
        <name>mapreduce.jobtracker.address</name>
        <value>MASTER-IP:54311</value>
    </property>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

#### File 5 : yarn-site.xml

* [x] Open yarn-site.xml file.

```
nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

* [x] Add following lines into yarn-site.xml file (Change MASTER-IP with the IP of your master node)

```
<configuration>
<!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>MASTER-IP</value>
    </property>
</configuration>
```

### **!!! Execute following steps on ONLY MASTER NODE**

* [x] Open masters file

```
nano /opt/hadoop-3.3.1/etc/hadoop/masters
```

* [x] Add master IP into file.(Change  IP)

```
160.75.19.160 
```

* [x] Open workers file.

```
nano /opt/hadoop-3.3.1/etc/hadoop/workers
```

* [x] Add IP's of workers into workers file.(CHANGE IP's)

```
160.75.19.162 
160.75.19.226
160.75.19.52
```

## Step 5 - Run Hadoop3

### **!!! Execute following steps on ONLY MASTER NODE**

* [x] We need to format namenode before starting hadoop (this is one time task do not run this more than one)

```
hdfs namenode -format
```

* [x] Start Distributed File System

```
start-dfs.sh
```

* [x] Start Yarn

```
start-yarn.sh
```

## Step 6 - Test Your Installation

* [x] Run jps command to see the status of your hadoop system.

```
jps
```

* [x] The output on Master.

```
18978 SecondaryNameNode
19092 Jps
18686 NameNode
```

* [x] The output on Workers.

```
14012 Jps
11242 DataNode
```

* [x] List Yarn nodes on master.

```
yarn node -list
```

## Step 7 - Run Hadoop & Yarn on Reboot

* [x] We will set cron job to run hadoop and yarn on reboot.

```
crontab -e
```

* [x] Add the following lines.

```
#Start Hadoop on Reboot
@reboot start-dfs.sh
@reboot start-yarn.sh
```

## Step 8 - Connect to UI's

* [x] For HADOOP

```
http://MASTER-IP:8088/cluster
```

* [x] For YARN

```
http://160.75.19.160:9870/dfshealth.html#tab-overview
```

## Congratulations :)



Your 3 Node Hadoop Cluster is up and running !!!
