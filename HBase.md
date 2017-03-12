## HBase

	HBase excels at providing key-based access to a specific cell of data, or a sequential range of cells.

	Denormalization,  Duplication,  and  Intelligent  Keys(DDI)

	each key is compared on a binary level, byte by byte, from left to right

	column family : 
	1. helps in building semantical or topical boundaries between the data, applying certain features(compression) to them, stored together in the same lowlevel storage file, HFile. 

	2. The name of the column family must be composed of printable characters, a notable difference from all other names or values.

	3. Column families need to be defined when the table is created and should not be changed too often, nor should there be too many of them.

	access data:
	(Table, RowKey, Family, Column, Timestamp) → Value

	regions:
	1. like range partitions in RDBMS
	
	2. one region could be split into two roughly equal halves by middle key.
	regions can ONLY be merged offline.
	
	3. 10  to  1,000 regions per server, but each between 1 GB and 2 GB in size. it all depends on what can be handled sufficiently by a single server

	HFiles:
	1. persistent and ordered immutable maps from keys to values. the files are sequences of blocks with a block index stored at the end. The index is loaded when the HFile is opened and kept in memory. The default block size is 64 KB but can be configured differently if required.

	data Update:
	1. first written to a commit log, called a  write-ahead log(WAL) in HBase, and then stored in the in-memory memstore. Once the data in memory has exceeded a given maximum value, it is flushed as an HFile to disk

	compaction:
	merges the files into larger ones
	1. minor compactions : reduce the number of storage files by rewriting smaller files into fewer but larger ones, performing an  n-way merge, merge is fast and bound only by disk I/O performance

	2. major compactions : rewrite all files within a column family for a region into a single new one. they can drop deleted entries including their deletion marker. Predicate deletes are handled here as well—for example, removing values that have expired according to the configured time-to-live or when there are too many versions

	master server:
	1. handling load balancing of regions across region servers. The master is not part of the actual data storage or retrieval path.


### Basic Configuration

	mounting disks with noatime option:
	/dev/sdd1 /data ext3 defaults,noatime 0 0

	ext4 delayed allocation:
	turn it off when using Hadoop

	XFS:
	There is a known shortcoming in the design that impacts metadata operations, such as deleting a large number of files

	ulimit:
	ulimit -n > 10000
	nproc

	/etc/sysctl.conf:
	fs.file-max
	vm.swappiness

	HDFS:
	dfs.datanode.max.xcievers > 4096

	Native Library checking:
	Make sure you have set up your Hadoop to use native, hardware checksumming. See link:[hadoop.native.lib].
	
	./bin/hbase --config ~/conf_hbase org.apache.hadoop.util.NativeLibraryChecker


### Column-based

	Reduced I/O
	the values of one column are often very similar in nature or even vary only slightly between logical rows, they are often much better suited for compression than the heterogeneous values of a row-oriented record structure

	
	delta and/or prefix compression : selected based on the type of the column can yield huge improvements in compression ratios. Better ratios result in more efficient bandwidth usage





### Commands

```shell
#Initialize hbase instance
${HBASE_HOME}/bin/start-hbase.sh

#Start a shell-like command-prompt
bin/hbase shell

#Basic actions:
#table_name				TN
#column_family_name		CFN
#column_name  			CN
#row_key 				RK
#column_value			VALUE
#namespace 				NS
hbase>create_namespace 'NS'
hbase>create 'TN', 'CFN' [in 'NS']
hbase>create '[NS:]TN', 'CFN'
hbase>list 'TN'
hbase>describe 'TN'
hbase>scan 'TN'
hbase>put 'TN', 'RK', 'CFN:CN', 'VALUE'
hbase>get 'TN', 'RK'
hbase>delete 'TN', 'RK', 'CFN:CN'
hbase>enable 'TN'

#Disable table before dropping it
hbase>disable 'TN'
hbase>drop 'TN'
hbase>drop_namespace 'NS'
```


### Client API

	HTable
	HTablePool


