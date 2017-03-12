## Spark Notes

	Application Monitoring @ http://<driver-node>:4040
	the Fair Scheduler

### Storage Configuration

	1.	no RAID
	2.	one mount point per physical disk
	3.	noatime mounting option for less disk writes

### Memory Allocation

-	allocating only at most 75% of the memory per node for Spark, leave the rest for the operating system and buffer cache

-	memory usage is greatly affected by storage level and serialization format

-	Java VM does not always behave well with more than 200 GB of RAM. If you purchase machines with more RAM than this, you can run multiple worker JVMs per node


### RDD

-	Transformatons => another RDD

-	Actions => result => driver program or external file

>each time we call a new action, the entire RDD must be
computed “from scratch.”

-	lazy fashion

>any RDD will not be computed until it reaches the first action.


-	RDD.persist

>in case the RDD will be used repeatedly

-	external data => RDD => transformation => (persistant) => actions


#### RDD Transformations

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


### Persistence

	SER: whether the data is serialized in memory, data un-serialization consumes CPU time.
	DISK : whether the data will be spilled on disk when there is too much data to fit in memory
	MEMORY_ONLY
	MEMORY_ONLY_SER
	MEMORY_AND_DISK
	MEMORY_AND_DISK_SER


	Cache policy:
	Don’t have to worry about job breaking if you ask Spark to cache too much data, but caching unnecessary data can lead to eviction of useful data and more recomputation time.



### Streaming

	groupId = org.apache.spark
	artifactId = spark-streaming_2.10
	version = 1.2.0


### Cluster Build

	driver : The central coordinator, runs in its own Java process, where the  main()method of your  program runs, it is the process running the user code that creates a SparkContext, creates RDDs, and performs transformations and actions
	1. Converting a user program into tasks
	2. Scheduling tasks on executors
		The driver also tracks the location of cached data tasks produced and uses it to schedule future tasks that access that data

	workers : executors, each executor is a separate Java process

	executors :
	1. run the tasks that make up the application and return results to the driver
	2. provide in-memory storage for RDDs that are cached by user programs, through a service called the Block Manager that lives within each executor

	tasks : physical execution, is the smallest unit of work in Spark, a typical user program can launch hundreds or thousands of individual tasks.


### Spark-submit

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



### Job Configuration

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
