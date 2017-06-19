# Spark Notes

	Application Monitoring @ http://<driver-node>:4040
	the Fair Scheduler

## Storage Configuration

	1.	no RAID
	2.	one mount point per physical disk
	3.	noatime mounting option for less disk writes

## Memory Allocation

-	allocating only at most 75% of the memory per node for Spark, leave the rest for the operating system and buffer cache

-	memory usage is greatly affected by storage level and serialization format

-	Java VM does not always behave well with more than 200 GB of RAM. If you purchase machines with more RAM than this, you can run multiple worker JVMs per node


## RDD

-	Transformatons => another RDD

-	Actions => result => driver program or external file

>each time we call a new action, the entire RDD must be
computed “from scratch.”

-	lazy fashion

>any RDD will not be computed until it reaches the first action.


-	RDD.persist

>in case the RDD will be used repeatedly

-	external data => RDD => transformation => (persistant) => actions


### RDD Transformations

	RDD.map(f) : applies the function f() to every element(per line usually in text mode) in RDD and KEEP the hierarchy of the RDD.

	RDD.flatMap(f) : same as RDD.map(), except FLATENing the hierarchy of the RDD.


	RDD.union(other RDD)

	RDD.intersection(other RDD)

	RDD.subtract(other RDD)

	RDD.cartesian(other RDD)

	RDD.filter(f)

	RDD.distinct()

	RDD.sample(withReplacement, fraction, [seed]) : RDD.sample(false,0.1)

### RDD Actions

	RDD.reduce(f) : returns the SAME type as of the element in input RDD.

	RDD.fold(f) : in addition takes a “zero value” to be used for the initial call on each partition. The zero value you provide should be the identity element for your operation

	What is the internal mechanism of fold?

	RDD.aggregate(initial_value, aggregator_function,combine_aggregator_function) : initial_value also fixes output frame.

	fiRDD.aggregate((0,0),
					(lambda acc, x : (acc[0] + x, acc[1] + 1)),
					(lambda acc1,acc2 : (acc1[0] + acc2[0], acc1[1] + acc2[1])))

	RDD.collect() : suffers from the restriction that all of your data must fit on a single machine, as it all needs to be copied to the driver

	RDD.take(n) : attempts to minimize the number of partitions it accesses, so it may represent a biased collection

	RDD.top(n) : use the default ordering on the data, but we can supply our own comparison function to extract the top elements.

	RDD.mean()

	RDD.variance()

	RDD.foreach(f)	: Run a function func on each element of the dataset. This is usually done for side effects such as updating an Accumulator or interacting with external storage systems.

### PairRDD

partitioning lets users control the layout of pair RDDs across nodes, Choosing the right partitioning for a distributed dataset is similar to choosing the right data structure for a local one—in both cases, data layout can greatly affect performance.

## Persistence

The first time the persisted RDD is computed in an action, it will be kept in memory on the nodes, RDD.unpersist() will manually remove the data from cache.

cache() method is a shorthand for using the default storage level, which is StorageLevel.MEMORY_ONLY (store deserialized objects in memory)

StorageLevel: provide different trade-offs between memory usage and CPU efficiency

In Python, stored objects will always be serialized with the Pickle library, so it does not matter whether you choose a serialized level.

```shell
SER: whether the data is serialized in memory, data un-serialization consumes CPU time, on the other hand, serialized data saves space
DISK : whether to spill partitions that don't fit in memory to disk instead of recomputing them on the fly each time they're needed.
_* : replicate each partition on * cluster nodes

**MEMORY_ONLY
MEMORY_ONLY_SER
**MEMORY_AND_DISK
MEMORY_AND_DISK_SER

MEMORY_ONLY_2
MEMORY_AND_DISK_2
etc

OFF_HEAP : delegate all the caching jobs to other distributed in-memory
file system, such as apache ignite or tachyon,
```

StorageLevel Strategy:

-	if using serialization, selecting a fast serialization library
-	Don’t spill to disk unless the functions that computed your datasets are expensive, recomputing a partition may be as fast as reading it from disk
-	Use the replicated storage levels if you want fast fault recovery (e.g. if using Spark to serve requests from a web application)


Cache policy:

	Don’t have to worry about job breaking if you ask Spark to cache too much data, but caching unnecessary data can lead to eviction of useful data and more recomputation time.


## Shared Variables

closure : The closure is those variables and methods which must be visible for the executor to perform its computations on the RDD, and the closure is serialized and sent to each executor. The executors running on seperate worker nodes each have their own copy of the closure.

constructs like loops or locally defined methods, should not be used to mutate some global state.

### Broadcast variables

keep a read-only variable cached on each machine rather than shipping a copy of it with tasks

explicitly creating broadcast variables is only useful when tasks across multiple stages need the same data or when caching the data in deserialized form is important.


### Accumulator

	Do NOT update accumulators in any transformations

Accumulator : used specifically to provide a mechanism for safely updating a variable when execution is split up across worker nodes in a cluster.

if accumulator updates performed **inside actions only**, Spark guarantees that each task’s update to the accumulator will only be applied once;

if accumulator updates performed **inside transformations**, each task’s update may be applied more than once if tasks or job stages are re-executed.


## Shuffle

The shuffle is Spark’s mechanism for re-distributing data so that it’s grouped differently across partitions, this typically involves copying data across executors and machines, making the shuffle a complex and costly operation.

-	mapPartitions to sort each partition using, for example, .sorted
-	repartitionAndSortWithinPartitions to efficiently sort partitions while simultaneously repartitioning
-	sortBy to make a globally ordered RDD

Repartition operations : repartition and coalesce
'ByKey operations : groupByKey and reduceByKey
Join operations : cogroup and join


## Streaming

ArtifactId in Maven:

	groupId = org.apache.spark
	artifactId = spark-streaming_2.10
	version = x.x.x


## Spark SQL

spark.sql.dialect : currently hiveql is a better choice, you have to use HiveContext, the default SQLContext provided by Spark only has 'sql' dialect in SQLContext.

### Interoperating with RDDs

#### Inferring the Schema Using Reflection

1.	define data schema(such as JavaBean)
2.	create an RDD of the type which fits the data schema
3.	using SQLContext(HiveContext) create a DataFrame from this RDD

#### Programmatically Specifying the Schema

1.	Create an RDD of Rows from the original RDD
2.	Create the schema represented by a StructType matching the structure of Rows in the RDD created in Step 1
3.	Apply the schema to the RDD of Rows via createDataFrame method provided by SQLContext(HiveContext)


### Data Sources

spark.sql.sources.default : default is parquet


#### Partition


### Schema Merging




## Cluster Build

	driver : The central coordinator, runs in its own Java process, where the  main()method of your  program runs, it is the process running the user code that creates a SparkContext, creates RDDs, and performs transformations and actions
	1. Converting a user program into tasks
	2. Scheduling tasks on executors
		The driver also tracks the location of cached data tasks produced and uses it to schedule future tasks that access that data

	workers : executors, each executor is a separate Java process

	executors :
	1. run the tasks that make up the application and return results to the driver
	2. provide in-memory storage for RDDs that are cached by user programs, through a service called the Block Manager that lives within each executor

	tasks : physical execution, is the smallest unit of work in Spark, a typical user program can launch hundreds or thousands of individual tasks.


## Spark-submit

```shell
bin/spark-submit [options] <app jar |python file> [app options]
# [options] : a list of flags for  spark-submit
#	--num-executors : default 2
#	--executor-memory : memory used by each executor
#	--executor-cores : number of cores a executor claims

# [app options] : application specific arguments
```

When submitting applications, options can be catagorized as :
1.	scheduling information
2.	runtime dependencies



## Job Configuration

spark-submit will automatically fill SparkConf for user when user-application pass an "empty" SparkConf to SparkContext

spark-submit will look for common SparkConf settings in conf/
spark-defaults.conf or any file indicated by --properties-file

SparkConf precedence order : SparkConf.set() > flags passed to spark-submit > values in the properties file > default values

Some Common Configs:

```shell
# default 512m, memory allocation per executor
spark.executor.memory
--executor-memory
```

```shell
# default 1; core allocation per executor
spark.executor.cores
--executor-cores
```

```shell
# default none; total cores for this job
spark.cores.max
--totalexecutor-cores
```

```shell
#default false; tasks that are running slowly will have a second copy launched on another node.Enabling this can help cut down on straggler tasks in large clusters.
spark.speculation
```


```shell
#default 45000ms; An internal timeout used for tracking the liveness of executors.
spark.storage.blockManagerTimeoutIntervalMs

```

```shell
# default "", customize the executor JVMs.
spark.executor.extraJavaOptions
spark.executor.extraClassPath
spark.executor.extraLibraryPath

```

```shell
# default org.apache.spark.serializer.JavaSerializer
# recommend org.apache.spark.serializer.KryoSerializer
spark.serializer
```

```shell
# default false, allows completed Spark jobs to be viewed using a history server
spark.eventLog.enabled

# default file:///tmp/sparkevents, The storage location used for event logging, if enabled. This needs to be in a globally visible filesystem such as HDFS
spark.eventLog.dir
```

```shell


```
```shell


```
```shell


```
```shell


```



## Spark Internals

+	Master
+	Driver
+	Worker
	*	ExecutorRunner
+	ExecutorBackend
	*	CoarsedGrainedExecutorBackend
		-	Executor
			+	Task

+	RDD dependency
	*	NarrowDependency - each partition of the parent RDD is used by at most one partition of the child RDD(只有前后两个 RDD 的 partition 个数以及 partitioner 都一样，才会出现 NarrowDependency)
		-	OneToOneDependency (1:1)
		-	NarrowDependency(N:1)
		-	NarrowDependency(N:N) - too few instance
		-	RangeDependency (只在 UnionRDD 中使用)
	*	ShuffleDependency

+	Pipeline
	*	Data only get computed when needed
	*	Data get "shuffled" to the location where computation happens

### 名词解释

RDD
Transformation
Action
Pipeline

