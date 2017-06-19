# Zookeeper Notes

bin/zkCli.sh -server 10.7.65.151:2181,10.7.65.152:2181,10.7.65.153:2181
bin/zkCli.sh -server 10.7.65.151:2181,10.7.65.152:2181,10.7.65.153:2181/test

## Basics

+   znodes : ZooKeeper data nodes which organized in hierarchical structure
+   ephemeral nodes : exists as long as the session that created the znode is active, can NOT have children
+   ensemble : a set of zookeeper servers
+   read-dominant
+   name space : a sequence of path elements separated by a slash (/),Every node in ZooKeeper's name space is identified by a path
+   watch : A watch solves problem for clients which have to poll constantly to acquire some message. It is set by client to be attached with one znode,the watch will be triggered and removed when the znode changes
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
+   quorum : minimum number of servers that have to store a client’s data before telling the client it is safely stored
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

```java
/**
 * We need to handle both ConnectionLossException and InterruptedException
 * when creating znode
 */

try{
    ...
    }catch(ConnectionLossException){
        ...
        }catch(InterruptedException){
            ...
        }
```

### Async Callback

```java
/**
 *  All synchronous calls in ZooKeeper have corresponding asynchronous calls,
 *  these calls dont't throw any KeeperException
 *
 *  @Param rc return code
 *  @Param ctx context object passed to the specific AsyncCallback call
 *  @Param stat Stat of the znode?
 */

Interface AsyncCallback
void processResult(int rc, ... , Object ctx, ... , Stat stat)




```




## Production Optimizations

+   A dedicated transaction log directory : By default transaction logs are put in $dataDir, change $dataLogDir to use for the transaction logs


## ACLs

+   ZooKeeper provides per-znode ACLs with a pluggable authentication method


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

