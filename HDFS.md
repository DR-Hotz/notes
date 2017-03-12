# HDFS


## A

	A  userspace  filesystem : the filesystem code runs outside the kernel as OS processes and by extension, is not registered with or exposed via the Linux VFS layer.


	Big data blocks minimizes drive seek operations, one of the slowest operations a mechanical disk can perform, results in better performance when doing large streaming I/O operations.


## fsimage && edit log

	edit log:
	it's a single entity, but also has segments on disk, every segment(single file) has a name like edits_TRANSACTION_ID format.

	fsimage:
	Each fsimagefile is a complete persistent checkpoint of the filesystem metadata(The suffix indicates the last transaction in the image),it is notupdated for every filesystem write operation

	fsimage  file  does  not  record  the  datanodes  on  which  the  blocks  are stored, Instead, the namenode keeps this mapping in memory, which it constructs by asking the datanodes for their block lists when they join the cluster and periodically afterward to ensure the namenode’s block mapping is up to date

	Write Operation:
	1.	edit log modified
	2.	in-memory metadata updated
	3.	


## secondary namenode

	produce check‐points of the primary’s in-memory filesystem metadata

## Administrative methods

### Commands

	hdfs dfsadmin -saveNamespace
	
### Parameters

	dfs.namenode.checkpoint.period : in seconds
	dfs.namenode.checkpoint.txns : transactions up limit
	dfs.namenode.checkpoint.check.period : in senconds
	dfs.namenode.checkpoint.dir
