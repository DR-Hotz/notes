## Hadoop Notes


### Map

#### Input

-	mapreduce.input.fileinputformat.input.dir.recursive

>true if the input directory is recursive.

-	mapreduce.input.fileinputformat.split.minsize

>minimum size of a split, default 1 byte

-	mapreduce.input.fileinputformat.split.maxsize

>maximum size of a split, default LONG.MAX_VALUE, this parameter has an effect only when it is less than the block size
max(minimumSize, min(maximumSize, blockSize))

### Shuffle and Sort

-	Sort order determination

>1. set mapreduce.job.output.key.comparator.class parameter
>2. keys must be a subclass of  WritableComparable,and the registered
comparator for the key class is used
>3. RawComparator, deserializes the byte streams being compared into objects and delegates to the WritableComparable’s compareTo()method




### Reduce

-	aims for reducers that each run for five minutes or so, and which
produce at least one HDFS block’s worth of output



### Output

-	 mapreduce.job.output.key.comparator.class

>set the key comparator class for sorting output, this could be set by calling
job.setSortComparatorClass()


-	




### Others

Metastore connection URL:	 jdbc:derby:;databaseName=metastore_db;create=true
Metastore Connection Driver :	 org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:	 APP
Starting metastore schema initialization to 2.1.0
Initialization script hive-schema-2.1.0.derby.sql
Initialization script completed
schemaTool completed
