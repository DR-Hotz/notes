# Zookeeper Notes

## Basics

+   znodes : ZooKeeper data nodes which organized in hierarchical structure
+   ephemeral nodes : exists as long as the session that created the znode is active, can NOT have children
+   ensemble : a set of zookeeper cluster
+   read-dominant
+   name space : a sequence of path elements separated by a slash (/),Every node in ZooKeeper's name space is identified by a path
+   watch : A watch is set by client to be attached with one znode, it will be triggered and removed when the znode changes
    *   One-time trigger
    *   Sent to the client : a client will never see a change for which it has set a watch until it first sees the watch event
    *   Data watches and child watches
+   event
    *   Created event
    *   Deleted event
    *   Changed event
    *   Child event
    *   Child Remove event
    *   Data Remove event
+   quorum : A replicated group of servers in the same application
+   quotas : tunable resources assigned to user.
+   time
    *   zxid : zookeeper transaction id
    *   version numbers : increases when znode changes
        -   version : number of changes to the data of a znode
        -   cversion : number of changes to the children of a znode
        -   aversion : number of changes to the ACL of a znode
    *   ticks : cluster zookeeper servers use ticks to define timing of event
    *   real time : ZooKeeper doesn't use real time
+   stat structure
    *   czxid : The zxid of the change that caused this znode to be created
    *   mzxid : The zxid of the change that last modified this znode
    *   ctime : 
    *   mtime : 
    *   version :
    *   cversion : 
    *   aversion :
    *   ephemeralOwner : 
    *   dataLength : The length of the data field of this znode
    *   numChildren : The number of children of this znode
+   Consistency Guarantees
    *   Sequential Consistency : Updates from a client will be applied in the order that they were sent
    *   Atomicity : Updates either succeed or fail -- there are no partial results
    *   Single System Image : A client will see the same view of the service regardless of the server that it connects to
    *   Reliability
    *   Timeliness
    *   The clients view of the system is guaranteed to be up-to-date within a certain time bound(on the order of tens of seconds)

## Watches && Event



## Programming with ZooKeeper

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.5.1-alpha</version>
</dependency>
```


## Production Optimizations

+   A dedicated transaction log directory : By default transaction logs are put in $dataDir, change $dataLogDir to use for the transaction logs


## Cluster Setup

### Configurations

```shell
#edit $ZOOKEEPER_HOME/conf/zoo.cfg
tickTime=2000
dataDir=/data/zookeeper
clientPort=2181
initLimit=5
syncLimit=2
#at least 3 servers
server.1=acct-pf1.hadoop.cib.com.cn:2888:3888
server.2=acct-pf2.hadoop.cib.com.cn:2888:3888
server.3=acct-pf3.hadoop.cib.com.cn:2888:3888

#Create myid file in $DataDir, store the identifier number in it
#Server.1
echo 1 > /data/zookeeper/myid

#Server.2
echo 2 > /data/zookeeper/myid

#Server.3
echo 3 > /data/zookeeper/myid

#All 3 servers set,then startup service
bin/zkServer.sh start
```
