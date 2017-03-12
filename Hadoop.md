# Hadoop Notes

```shell
# Default configuration files

$HADOOP_INSTALL_DIR/share/doc/hadoop/hadoop-project-dist/had‌​oop-common/core-defa‌​ult.xml
$HADOOP_INSTALL_DIR/share/doc/hadoop/hadoop-project-dist/had‌​oop-hdfs/hdfs-defa‌​ult.xml
$HADOOP_INSTALL_DIR/share/doc/hadoop/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml
$HADOOP_INSTALL_DIR/share/doc/hadoop/hadoop-yarn/hadoop-yarn-common/yarn-default.xml

netstat -tuplen

systemctl get-default

systemctl set-default multi-user.target
graphical.target

```

## How Mapreduce works?

### Job Submission

1.	asks resource manager for new applicatin id(job id)

2.	checks output specifications

3.	computes input splits

4.	copies the resources needed to run the job to the shared file system

5.  calling submitApplication() to submit the job to resource manager

### Job Initialization

1.  YARN scheduler allocates container for application master

2.  resource manager launches app master's processes

3.  app master decides how to run the task

### Task Assignment

1.  app master requests containers for map and reduce tasks

### Task Execution

1.  resources allocated, app master starts container by contacting node manager

2.  task is executed by a Java app whose main class is YarnChild, localize the resources that task needs


## Useful parameters

-	io.file.buffer.size

>Hadoop buffer size, default 4kb

-	mapred.child.java.opts

>The amount of memory given to the JVMs in which the map and reduce tasks run.

-	mapreduce.map.maxattempts

-	mapreduce.map.failures.maxpercent

-	mapreduce.reduce.failures.maxpercent

-	mapreduce.task.io.sort.mb

>each map's circular memory buffer, 100MB default

-	mapreduce.task.io.sort.factor

>how many map outputs will be merged into one per round.

-	mapreduce.map.sort.spill.percent

>map'circular memory buffer reach certain threshold, background thread spills the contents to disk, 80% default.

-	mapreduce.cluster.local.dir

>where the spilled contents of map stored on disk.

-	 mapreduce.map.combine.minspills

>where the spill files are less the threashold, the combiner is not invoking.

-	 mapreduce.map.output.compress

>whether compress the map output, false by default, but compression is highly recommended.

-	mapreduce.map.output.compress.codec

>map output compression library.

-	mapreduce.shuffle.max.threads

>controls the maximum worker threads which serve the file partitions to reducer. Note that this setting is per node manager, not per map
task. The default of 0 sets the maximum number of threads to twice the number of processors on the machine.

-	mapreduce.reduce.input.buffer.percent

>???

-	mapreduce.reduce.shuffle.parallelcopies

>default 5 threads to fetch map outputs in parallel.

-	mapreduce.reduce.shuffle.input.buffer.percent

>dictates the proportion of the heap in reduce JVM which could be used to hold map output if the ouputs are small enough,otherwise, the map outputs are copied to disk.

-	mapreduce.reduce.shuffle.merge.percent

>threashold of the buffer before spilling outputs to disk.

-	mapreduce.reduce.merge.inmem.threshold

>threashold of the number of map outputs merged, after it's reached, reduce outputs is spill to disk, default to 0 means threads are twice as many as processes at reduce node.


## Map

The best performance can be obtained by avoiding multiple spills to
disk. mapreduce.task.io.sort.* property settings

-	SPILLED_RECORDS

>a MapReduce counter that counts the total number of records that were spilled to disk over the course of a job. Note that the counter includes both map- and reduce-side spills.

-	mapreduce.input.linerecordreader.line.maxlength

>max length per line in bytes.





## Shuffle and Sort

The process by which the system performs the sort—and transfers the map outputs to the reducers as inputs—is known as the shuffle

The general principle is to give the shuffle as much memory as possible

it is best to write your map and reduce functions to use as little memory as possible, avoid accumulating values in a map


## Reduce

copy phase -> sort phase(merge phase) -> reduce phase(mixture of in-memory and on-disk)

The best performance is obtained when the intermediate data can
reside entirely in memory.
