# hadoop cluster

https://www.linode.com/docs/guides/how-to-install-and-set-up-hadoop-cluster/

nano /etc/hosts

```bash
192.0.2.1    node-master
192.0.2.2    node1
192.0.2.3    node2
```

## genkey

```bash
ssh-keygen -b 4096

less /home/hadoop/.ssh/id_rsa.pub

cat ~/.ssh/master.pub >> ~/.ssh/authorized_keys
```

## download

```
cd
wget http://apache.cs.utah.edu/hadoop/common/current/hadoop-3.1.2.tar.gz
tar -xzf hadoop-3.1.2.tar.gz
mv hadoop-3.1.2 hadoop


nano /home/hadoop/.profile
PATH=/home/hadoop/hadoop/bin:/home/hadoop/hadoop/sbin:$PATH

nano /home/hadoop/.bashrc
export HADOOP_HOME=/home/hadoop/hadoop
export PATH=${PATH}:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
```

## Configure the Master Node

node-master

update-alternatives --display java

```
export JAVA_HOME=${JAVA_HOME}
```

nano ~/hadoop/etc/hadoop/hadoop-env.sh
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre
```

## Set NameNode Location

nano ~/hadoop/etc/hadoop/core-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>fs.default.name</name>
            <value>hdfs://node-master:9000</value>
        </property>
    </configuration>
```

nano ~/hadoop/etc/hadoop/hdfs-site.xml

```xml
<configuration>
    <property>
            <name>dfs.namenode.name.dir</name>
            <value>/home/hadoop/data/nameNode</value>
    </property>

    <property>
            <name>dfs.datanode.data.dir</name>
            <value>/home/hadoop/data/dataNode</value>
    </property>

    <property>
            <name>dfs.replication</name>
            <value>1</value>
    </property>
</configuration>
```


## Set YARN as Job Scheduler

nano ~/hadoop/etc/hadoop/mapred-site.xml

```xml
<configuration>
    <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
    </property>
    <property>
            <name>yarn.app.mapreduce.am.env</name>
            <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
    <property>
            <name>mapreduce.map.env</name>
            <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
    <property>
            <name>mapreduce.reduce.env</name>
            <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
</configuration>
```

## Configure YARN

nano ~/hadoop/etc/hadoop/yarn-site.xml

```xml
<configuration>
    <property>
            <name>yarn.acl.enable</name>
            <value>0</value>
    </property>

    <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>203.0.113.0</value>
    </property>

    <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

## Configure Workers

nano ~/hadoop/etc/hadoop/workers

```
node1
node2
```

(/home/hadoop/hadoop/etc/hadoop/yarn-site.xml)
nano ~/hadoop/etc/hadoop/yarn-site.xml

```xml
<property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>1536</value>
</property>

<property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>1536</value>
</property>

<property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>128</value>
</property>

<property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
</property>
```

(/home/hadoop/hadoop/etc/hadoop/mapred-site.xml)
nano ~/hadoop/etc/hadoop/mapred-site.xml

```xml
<property>
        <name>yarn.app.mapreduce.am.resource.mb</name>
        <value>512</value>
</property>

<property>
        <name>mapreduce.map.memory.mb</name>
        <value>256</value>
</property>

<property>
        <name>mapreduce.reduce.memory.mb</name>
        <value>256</value>
</property>
```

## Duplicate Config Files on Each Node

1.Copy the Hadoop binaries to worker nodes:

```bash
cd /home/hadoop/
scp hadoop-*.tar.gz node1:/home/hadoop
scp hadoop-*.tar.gz node2:/home/hadoop
```

2.Connect to node1 via SSH. A password isn’t required, thanks to the SSH keys copied above:

```bash
ssh node1
```

3.Unzip the binaries, rename the directory, and exit node1 to get back on the node-master:

```bash
tar -xzf hadoop-3.1.2.tar.gz
mv hadoop-3.1.2 hadoop
exit
```

4.Repeat steps 2 and 3 for node2.

5.Copy the Hadoop configuration files to the worker nodes:

```bash
for node in node1 node2; do
    scp ~/hadoop/etc/hadoop/* $node:/home/hadoop/hadoop/etc/hadoop/;
done
```

## Format HDFS

HDFS needs to be formatted like any classical file system. On node-master, run the following command:

```bash
hdfs namenode -format
```

# Run and monitor HDFS

This section will walk through starting HDFS on NameNode and DataNodes, and monitoring that everything is properly working and interacting with HDFS data.


## Start and Stop HDFS

1.Start the HDFS by running the following script from node-master:

```bash
start-dfs.sh
```

This will start `NameNode` and `SecondaryNameNode` on `node-master`, and `DataNode` on `node1` and `node2` , according to the configuration in the `workers` config file.

2.Check that every process is running with the jps command on each node. On `node-master`, you should see the following (the PID number will be different):

```bash
21922 Jps
21603 NameNode
21787 SecondaryNameNode
```

And on node1 and node2 you should see the following:

```bash
19728 DataNode
19819 Jps
```

3.To stop HDFS on master and worker nodes, run the following command from node-master:

```bash
stop-dfs.sh
```

## Monitor your HDFS Cluster

1.You can get useful information about running your HDFS cluster with the hdfs dfsadmin command. Try for example:

```bash
hdfs dfsadmin -report
```

This will print information (e.g., capacity and usage) for all running DataNodes. To get the description of all available commands, type:

```bash
hdfs dfsadmin -help
```

2.You can also automatically use the friendlier web user interface. Point your browser to http://node-master-IP:9870, where node-master-IP is the IP address of your node-master, and you’ll get a user-friendly monitoring console.

## Put and Get Data to HDFS

Writing and reading to HDFS is done with command hdfs dfs. First, manually create your home directory. All other commands will use a path relative to this default home directory:

```bash
hdfs dfs -mkdir -p /user/hadoop
```

1.Create a books directory in HDFS. The following command will create it in the home directory, /user/hadoop/books:

```bash
hdfs dfs -mkdir books
```

2.Grab a few books from the Gutenberg project:

```bash
cd /home/hadoop
wget -O alice.txt https://www.gutenberg.org/files/11/11-0.txt
wget -O holmes.txt https://www.gutenberg.org/files/1661/1661-0.txt
wget -O frankenstein.txt https://www.gutenberg.org/files/84/84-0.txt
```

3.Put the three books through HDFS, in the booksdirectory:

```bash
hdfs dfs -put alice.txt holmes.txt frankenstein.txt books
```

4.List the contents of the book directory:

```bash
hdfs dfs -ls books
```

5.Move one of the books to the local filesystem:

```bash
hdfs dfs -get books/alice.txt
```

6.You can also directly print the books from HDFS:

```bash
hdfs dfs -cat books/alice.txt
```

https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html

```bash
hdfs dfs -help
```

## Run YARN

### Start and Stop YARN

1.Start YARN with the script:

```bash
start-yarn.sh
```

2.Check that everything is running with the `jps` command. In addition to the previous HDFS daemon, you should see a `ResourceManager` on `node-master`, and a `NodeManager` on `node1` and `node2`

3.To stop YARN, run the following command on node-master:

```bash
stop-yarn.sh
```

## Monitor YARN

1.The yarn command provides utilities to manage your YARN cluster. You can also print a report of running nodes with the command:

```bash
yarn node -list
```

Similarly, you can get a list of running applications with command:

```bash
yarn application -list
```

https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YarnCommands.html


2.As with HDFS, YARN provides a friendlier web UI, started by default on port 8088 of the Resource Manager. Point your browser to `http://node-master-IP:8088` where node-master-IP is the IP address of your node-master, and browse the UI:

## Submit MapReduce Jobs to YARN

YARN jobs are packaged into jar files and submitted to YARN for execution with the command yarn jar. The Hadoop installation package provides sample applications that can be run to test your cluster. You’ll use them to run a word count on the three books previously uploaded to HDFS.

1.Submit a job with the sample jar to YARN. On node-master, run:

```bash
yarn jar ~/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.2.jar wordcount "books/*" output
```

The last argument is where the output of the job will be saved - in HDFS.

2.After the job is finished, you can get the result by querying HDFS with hdfs dfs -ls output. In case of a success, the output will resemble:

```bash
Found 2 items
-rw-r--r--   2 hadoop supergroup          0 2019-05-31 17:21 output/_SUCCESS
-rw-r--r--   2 hadoop supergroup     789726 2019-05-31 17:21 output/part-r-00000
```

3.Print the result with:

```bash
hdfs dfs -cat output/part-r-00000 | less
```

