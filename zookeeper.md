# Zookeeper Notes

## Basics

+   znodes : ZooKeeper data nodes which organized in hierarchical structure
+   ephemeral nodes : exists as long as the session that created the znode is active
+   ensemble
+   read-dominant
+   name space : a sequence of path elements separated by a slash (/),Every node in ZooKeeper's name space is identified by a path
+   watches : A watch is set by client to be attached with one znode, it will be triggered and removed when the znode changes
+   quorum : A replicated group of servers in the same application
+   Quotas : tunable resources assigned to user.

## Programming with ZooKeeper




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
