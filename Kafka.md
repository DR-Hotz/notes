# Kafka Notes

```shell
#Producer && Consumer:
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>0.10.2.0</version>
    </dependency>
#Streams :
    <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>0.10.2.0</version>
    </dependency>
#
```


## Basics

+   Broker : Kafka Server
+   Producer : allows applications to send streams of data to topics in the Kafka cluster
+   Connector : an extensible tool which implement the custom logic for interacting with an external system, it could continually pull from some source data system into Kafka or push from Kafka into some sink data system
+   Consumer : allows applications to read streams of data from topics in the Kafka cluster
+   Stream Processor : allows transforming streams of data from input topics to output topics

+   Topic
    *   Partitions
    *   Leader
    *   Replication Factor
        -   Replicas
        -   isr

+   Push && pull
    *   


### Command List

+   Topic
```shell
#Create topic
bin/kafka-topics.sh --create --zookeeper {hostname:port,hostname:port} --replication-factor {} --partitions {} --topic {topic_name}

bin/kafka-topics.sh --delete --zookeeper {hostname:port,hostname:port} --topic {topic_name}

bin/kafka-topics.sh --alter

bin/kafka-topics.sh --list --zookeeper {hostname:port,hostname:port}
e.g: bin/kafka-topics.sh --list --zookeeper 10.7.65.151:2181,10.7.65.152:2181,10.7.65.153:2181
```

+   Producer
```shell


```

## Guarantees

+   Within any producer's process, records which sent to topic are as in the same order as they are sent by the producer.
+   A consumer instance sees records in the order they are stored in the log
+   For a topic with replication factor N, we will tolerate up to N-1 server failures without losing any records committed to the log

### Message Delivery Guarantees

+   At most once : Messages may be lost but are never redelivered.
+   At least once : Messages are never lost but may be redelivered.
+   Exactly once : each message is delivered once and only once.

## Performance

### Producer

+   All Kafka nodes can answer a request for metadata about which servers are alive and where the leaders for the partitions of a topic are at any given time
+   Partition
+   Batching : attempt to accumulate data in memory and to send out larger batches in a single request, This buffering is configurable and gives a mechanism to trade off a small amount of additional latency for better throughput



### Consumer

+   topic is divided into a set of totally ordered partitions, each of which is consumed by exactly one consumer within each subscribing consumer group at any given time
+   Offline Data Load



## Log Compaction


## Quotas


## Topic

+   Create a topic
```shell
bin/kafka-topics.sh --create --zookeeper {host:port} --replication-factor {num} --partitions {num} --topic {name}

```

## Tunings

### Broker


### Producer

+   bootstrap.servers

### Consumer

## API

### Producer

+   Handle queueing/buffering of multiple producer requests and asynchronous dispatch of the batched data
+   Handles the serialization of data through a user-specified Encoder
```java
 interface Encoder<T> {
    public Message toMessage(T data);
    }
```

+   provides software load balancing through an optionally user-specified Partitioner
```java

```

```shell


```


## Crucial Parameters

### Broker
+   broker.id : a unique identifier of kafka cluster node(broker)
+   log.dirs : where kafka logs are stored, which could be a list of directories.
+   zookeeper.connect : a list of comma-separated zookeeper cluster address
+   host.name
+   num.partitions : default topic partitions number
+   auto.create.topics.enable : whether allowing automatic topic creation
+   default.replication.factor : as the name indicates.

### Producer
+   max.block.ms : If records are sent faster than they can be transmitted to the server then this buffer space will be **exhausted**. When the buffer space is exhausted additional send calls will **block**. The threshold for time to block is determined by **max.block.ms** after which it throws a TimeoutException
+   bootstrap.servers : A list of host/port pairs to use for establishing the initial connection to the Kafka cluster
+   key.serializer/value.serializer
+   acks
    *   0 - producer won't wait for any ack
    *   1 - leader will send ack right after receiving the message rather than waiting for other followers to get the replicas
    *   -1/all - leader will wait for completion of all the followers and then send ack
+   buffer.memory : total amount of memory available to the producer for buffering
+   compression.type
    *   none/gzip/snappy/lz4
+   retries : enabling retries also opens up the possibility of duplicates
+   batch.size : buffers of **unsent records for each partition**, attempt to batch records together into fewer requests whenever multiple records are being sent to the same partition
+   linger.ms : how long to wait until the unsent messages in buffer(per partition) get sent, Note that under heavy load batching will occur regardless of the linger configuration

## Cluster Setup

+   Setup each server configs
```shell
#edit each broker's $KAFKA_HOME/config/server.properties
#each broker has a unique id
broker.id=0/1/2
log.dirs=/data/kafka-logs
listeners=PLAINTEXT://:9092
num.partitions=5

#zookeeper service, you have to create /kafka node first
zookeeper.connect=acct-pf1.hadoop.cib.com.cn:2181,acct-pf2.hadoop.cib.com.cn:2181,acct-pf3.hadoop.cib.com.cn:2181/kafka

#start each broker
nohup bin/kafka-server-start.sh config/server.properties 2>&1 >/dev/null &
```

+   Tuning Cluster
```shell


```

### Topic Setup


