# Hadoop集群搭建

```shell
git config --global core.autocrlf true
git config --global core.autocrlf input
```


## 1. 基础配置

### 1.1 磁盘划分

	Hadoop集群数据存储磁盘采用JBOD(Just a Bunch Of Disks)挂载方式，不采用RAID,每一块物理盘映射一个挂载点，文件系统采用ext4 或 xfs

	#/etc/fstab配置含义如下:
	#<fs>                  <mountpoint>   <type>   <opts>     <dump/pass>
	#fs: 设备所在路径
	#mountpoint: 挂载点，注意在挂载前一定要先建立该路径
	#type: 文件系统类型
	#opts: 挂载选项，如
	#      noatime - 关闭atime特性
	#      defaults - 等价于rw,suid,dev,exec,auto,nouser,async
	#      auto/noauto - 自动/手动挂载
	#      exec/noexec - 二进制文件可以/禁止执行
	#      user/nouser - 允许任何用户挂载/只能root挂载
	#      suid/nosuid - 允许/禁止suid,sgid产生作用,主要用于临时授权执行二进制文件
	#dump: 0-不备份  1-备份
	#pass: fsck检查 0-不检查  1-最高优先权  2-其他所有的设备为2

```shell
fdisk /dev/sdb
#进入fdisk界面，选择如下
#1. n 						add a new partition
#2. p 						primary partition
#3. 1 						partition number
#4. 默认回车 				first sector
#5. 默认回车				last sector
#6. 返回初始界面后选择t 	change a partition's system id
#7. 83						Linux
#8. p 						确认当前划分整个磁盘并且类型是Linux
#9. w 						确认变更

#之后键入如下命令
partprobe

#创建文件系统并配置自动挂载
mkfs -t ext4 /dev/sdb1

#配置/etc/fstab,添加如下行,保存并退出
/dev/sdb1	/data 	ext4	defaults,noatime	0 0

#建立挂载点
mkdir /data

#挂载磁盘
mount -a
```

### 1.2 其他配置

```shell
#配置本机host name，集群每台机器都配置，注意替换hostname中的数字
#编辑/etc/sysconf/network,添加如下内容:
HOSTNAME=acct-pf1.hadoop.cib.com.cn


#配置集群hosts
#编辑/etc/hosts,添加如下内容:
10.7.65.151	acct-pf1.hadoop.cib.com.cn
10.7.65.152	acct-pf2.hadoop.cib.com.cn
10.7.65.153	acct-pf3.hadoop.cib.com.cn
10.7.65.154	acct-pf4.hadoop.cib.com.cn
10.7.65.155	acct-pf5.hadoop.cib.com.cn
10.7.65.156	acct-pf6.hadoop.cib.com.cn
```

```shell
#关停SELinux
#编辑/etc/selinux/config
SELINUX=disabled

#重启
reboot -f
```

### 1.3 NTP 配置

```shell
#如无ntpd服务，则先安装
rpm -Uvh ntp-4.2.6p5-22.el7.x86_64.rpm

#编辑/etc/ntp.conf，注释掉外网server，添加如下server
#10.7.65.149为配置ntp server的主机IP地址
server 10.7.65.149

#重启ntpd
service nptd restart

#查看ntpd服务是否正常
serivce nptd status
ntpq -p
```


## 2. 用户 && 组配置

```shell
groupadd -g 801 hadoop

useradd -m -u 801 -g hadoop -p hadoop hadoop
useradd -m -u 802 -g hadoop -p yarn yarn
useradd -m -u 803 -g hadoop -p hdfs hdfs
useradd -m -u 804 -g hadoop -p spark spark


#将所有存储挂载点的所有权都给hadoop用户,本例中目前只有一个/data
chown -R hadoop:hadoop /data
chmod 775 /data

#ssh配置
#本机对本机的ssh也要加入authorized_keys

#生成ssh秘钥对,并配置authorized_keys
ssh-keygen

#hadoop hdfs yarn用户在集群之间均要建立互信关系
```


## 3. Java 安装

```shell
#将linux x86-64版本java 1.7 安装在/usr/lib/java-1.7.0下
```

## 4. Python 安装

	在当前RHEL 6.7 版本上的Python版本为2.6.6，而spark 要求python版本最好为2.7+，需要将每台节点机器的python版本升级为2.7+

	编译LTO优化过的Python需要一套带LTO、PGO功能的编译工具套件，可在一台机器上先准备该工具套件，在保证节点每台机器都安装有Python运行时依赖包的情况下，可只在一台机器上编译好高性能的Python，之后安装至每台机器，因此4.1、4.3两节可以在一台机器上完成，4.2Python运行时依赖的包则必须在所有节点上事先安装，由于测试环境OS环境与生产环境的包并不完全相同，因此4.2节中Python的运行时依赖可能更多，准备OS原版镜像安装即可

	本节安装需要两个大包文件 compile.tgz 和 python27.tgz

### 4.1 工具准备

	Python 2.7的 --enable-optimizations编译选项采用LTO，要求一定版本编译套件支持


```shell
#先用gcc4.4.7编译一个binutils版本用与其他编译
#编译安装GNU binutils 2.28版本
#首先创建一个与binutils的source directory同级别的文件夹binutils-build
mkdir binutils-build
cd binutils-build
../binutils-2.28/configure --prefix=/usr/local --build=x86_64-redhat-linux --enable-lto
make
make check
make install

#安装好后通过which确认下ld、as等命令是否更新
#ld、as此时路径应为/usr/local/bin
which ld  
which as
```

```shell
#GCC 4.9.4
#如果需要gcc编译出32bit版本软件，则需安装32位的libc及其headers,这里禁止了该功能
#与之前安装binutils一样，此时在gcc源码文件夹同层建立一个空的文件夹gcc-build
mkdir gcc-build
cd gcc-build
../gcc-4.9.4/configure --prefix=/usr/local --disable-multilib --enable-lto \
--enable-languages=c,c++ --enable-shared --enable-threads=posix \
--build=x86_64-redhat-linux --with-ld=/usr/local/bin/ld \
--with-as=/usr/local/bin/as

make
make check
make install

#安装好后确保默认的gcc已经指向/usr/local/bin下
which gcc
```

### 4.2 依赖准备

	以下只是目前环境中缺失的依赖，如果在编译中还缺失其他依赖，可在OS安装镜像中找到

	tcl-8.5.7-6.el6.x86_64.rpm
	tcl-devel-8.5.7-6.el6.x86_64.rpm
	tk-8.5.7-5.el6.x86_64.rpm
	tk-devel-8.5.7-5.el6.x86_64.rpm
	tix-8.4.3-5.el6.x86_64.rpm
	tkinter-2.6.6-64.el6.x86_64.rpm


```shell
#如果不确定少了哪些rpm依赖，可在我预先打包的python-build下找到
#一份rpms.list, 这份清单列出了本次编译安装Python时系统已经安装了的rpms包
#可将这些包一并安装即可

rpm -Uvh tcl-8.5.7-6.el6.x86_64.rpm
rpm -Uvh tcl-devel-8.5.7-6.el6.x86_64.rpm
rpm -Uvh tk-8.5.7-5.el6.x86_64.rpm
rpm -Uvh tk-devel-8.5.7-5.el6.x86_64.rpm
rpm -Uvh tix-8.4.3-5.el6.x86_64.rpm
rpm -Uvh tkinter-2.6.6-64.el6.x86_64.rpm
```


### 4.3 编译安装

重新以root用户登录

```shell
#python-build文件夹与Pyhthon源码文件夹必须处于同一层级
mkdir python-build
cd python-build
../Python-2.7.13/configure --prefix=/usr/local --enable-optimizations --enable-shared --build=x86_64-redhat-linux-gnu

make
#在make之后，注意缺少的安装包提示，本次make缺少的依赖如下所示：

```
	Python build finished, but the necessary bits to build these modules were not found:
	bsddb185           dl                 imageop         
	sunaudiodev

```shell
#以上缺少的几个包都不是必须，有些例如sunaudiodev等不是x86_64 Linux平台的包，可以忽略

#务必确认所有Python找不到的依赖包都是不需要的包之后再进行安装
make install


#添加运行时依赖
#编辑/etc/ld.so.conf文件，添加如下两行
/usr/local/lib
/usr/local/lib64

#运行ldconfig添加运行时库
#如果出现"xxx.py is not an ELF file" 提示，不必理会
ldconfig

#运行python检查版本
python --version
```


```shell
#在其他非编译的机器上安装时，可将编译好的包python-release覆盖到相应目录即可
cd python-release/usr/local
cp -R * /usr/local



#建立软链接
cd /usr/local/bin

ln -sf python2.7-config python2-config
ln -sf python2-config python-config  
ln -sf python2.7 python2
ln -sf python2 python

cd /usr/local/lib/pkgconfig
ln -sf python-2.7.pc python2.pc
ln -sf python2.pc python.pc

```

### 4.4 其他安装

```shell
#先安装setuptools，由于setuptools34 安装时会出现互相依赖问题
#因此先安装30，之后有pip之后再升级至34即可
tar -xzf setuptools-30.0.0.tar.gz
cd setuptools-30.0.0
python setup.py install
```


```shell
#安装pip
tar -xzf pip-9.0.1.tar.gz
cd pip-9.0.1
python setup.py install

#安装好pip之后，就可以升级setuptools了
#准备所有的离线安装包在python27/packages下，有pip之后，可按如下流程安装其他包
#只要所有的依赖都在--find-links文件夹内，pip会自动安装依赖
pip install --no-index --find-links=/data/packs/python27/packages setuptools-34.1.1-py2.py3-none-any.whl
```

```shell
#安装其他的常用的python依赖
pip install --no-index --find-links=/data/packs/python27/packages \
numpy-1.12.0-cp27-cp27m-manylinux1_x86_64.whl \
scipy-0.18.1-cp27-cp27m-manylinux1_x86_64.whl \
virtualenv-15.1.0-py2.py3-none-any.whl

```

## 5. 集群安装 && 配置

```shell
#配置所有用户的环境变量
#在/etc/profile.d下添加脚本custom.sh,内容如下

#新建/etc/profile.d/custom.sh,设置全局环境变量
JAVA_HOME=/usr/lib/java-1.7.0/jdk1.7.0_80
HADOOP_PREFIX=/usr/local/hadoop-2.7.3
HADOOP_CONF_DIR=/usr/local/etc/hadoop
HADOOP_LOG_DIR=/data/hadoop-logs
YARN_LOG_DIR=/data/yarn-logs
SPARK_HOME=/usr/local/spark-1.5.1-bin-without-hadoop
PATH=/opt/binutils/bin:$JAVA_HOME/bin:$HADOOP_PREFIX/sbin:$HADOOP_PREFIX/bin:$SPARK_HOME/sbin:$SPARK_HOME/bin:$PATH
LD_LIBRARY_PATH=/usr/local/hadoop-2.7.3/lib/native:$LD_LIBRARY_PATH

export JAVA_HOME HADOOP_PREFIX HADOOP_CONF_DIR HADOOP_LOG_DIR YARN_LOG_DIR SPARK_HOME PATH LD_LIBRARY_PATH

alias l='ls -lart'
```

### 5.1 Hadoop 安装 && 配置

	本次安装采用hadoop-2.7.3

```shell
#将安装包解压至/usr/local下

chown -R hadoop:hadoop /usr/local/hadoop-2.7.3


#配置/etc/hadoop/slaves,每一行添加一个data node
acct-pf2.hadoop.cib.com.cn
acct-pf3.hadoop.cib.com.cn
acct-pf4.hadoop.cib.com.cn
acct-pf5.hadoop.cib.com.cn
acct-pf6.hadoop.cib.com.cn

```

将每个节点的${HADOOP_PREFIX}/etc/hadoop下的配置资料全部复制到/usr/local/etc/hadoop下，分别修改以下文件内容

```shell
#core-site.xml 配置参数如下

<property>
	<name>fs.defaultFS</name>
	<value>hdfs://acct-pf1.hadoop.cib.com.cn:9527</value>
</property>
<property>
	<name>hadoop.tmp.dir</name>
	<value>/data/tmp</value>
</property>
<property>
	<name>io.file.buffer.size</name>
	<value>131072</value>
</property>

```

```shell
#yarn-site.xml 配置

#Default web port of yarn is 8088
<property>
	<name>yarn.resourcemanager.hostname</name>
	<value>acct-pf1.hadoop.cib.com.cn</value>
</property>

<!-- 用来存放 datanode 上的临时文件,多路径可分散IO -->
<property>
	<name>yarn.nodemanager.local-dirs</name>
	<value>/data/nm,/data1/nm</value>
</property>
<property>
	<name>yarn.nodemanager.log-dirs</name>
	<value>/data/nm-logs,/data1/nm-logs</value>
</property>
<property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>20480</value>
</property>
<property>
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
</property>

```

```shell
#hdfs-site.xml 配置参数

#namenode 镜像至少2个磁盘,最好再放个NFS
<property>
	<name>dfs.namenode.name.dir</name>
	<value>file:///data/nn,file:///data1/nn</value>
	<final>true</final>
</property>
<property>
	<name>dfs.blocksize</name>
	<value>268435456</value>
</property>

<!-- 搭建开发环境，设置为2即可 -->
<property>
  <name>dfs.replication</name>
  <value>2</value>
</property>
<property>
	<name>dfs.permissions.superusergroup</name>
	<value>hadoop</value>
</property>
<!-- 所有datanode 节点列表文件 -->
<property>
	<name>dfs.hosts</name>
	<value>/etc/hadoop/slaves</value>
</property>

<!-- 注意每个data node都要有以下mount point -->
<property>
	<name>dfs.datanode.data.dir</name>
	<value>/data/dn</value>
</property>
```


```shell
#mapred-site.xml 配置

<property>
	<name>mapreduce.framework.name</name>
	<value>yarn</value>
</property>
<property>
	<name>mapreduce.task.io.sort.mb</name>
	<value>256</value>
</property>
<property>
	<name>mapreduce.jobhistory.address</name>
	<value>acct-pf1.hadoop.cib.com.cn:10020</value>
</property>
<property>
	<name>mapreduce.jobhistory.webapp.address</name>
	<value>acct-pf1.hadoop.cib.com.cn:19888</value>
</property>
<property>
	<name>mapreduce.jobhistory.intermediate-done-dir</name>
	<value>/data/mr-history/tmp</value>
</property>
<property>
	<name>mapreduce.jobhistory.done-dir</name>
	<value>/data/mr-history/done</value>
</property>
```


### 5.2 Spark 安装 && 配置

	本次安装采用不带Hadoop包的独立Spark 1.5.1 版本

```shell
#解压缩安装包至/usr/local

#改变属主
chown -R hadoop:hadoop /usr/local/spark-1.5.1-bin-without-hadoop

#编辑${SPARK_HOME}/conf下的spark-env.sh脚本
#首先从模板创建一个
cp spark-env.sh.template spark-env.sh

#编辑spark-env.sh，添加如下配置
HADOOP_CONF_DIR=/usr/local/etc/hadoop
SPARK_LOCAL_DIRS=/data/spark-local-dir
SPARK_LOG_DIR=/data/spark-logs
SPARK_DIST_CLASSPATH=$(hadoop classpath)

export HADOOP_CONF_DIR SPARK_LOCAL_DIRS SPARK_LOG_DIR SPARK_DIST_CLASSPATH

#编辑${SPARK_HOME}/conf下的slaves
#首先从模板创建一个
cp slaves.template slaves


#将${SPARK_HOME}/conf下的spark-env.sh脚本copy到每个集群每个节点上
cd /usr/local/spark-1.5.1-bin-without-hadoop/conf
scp spark-env.sh acct-pf2.hadoop.cib.com.cn:/usr/local/spark-1.5.1-bin-without-hadoop/conf

#运行Spark集群
cd ${SPARK_HOME}/sbin
./start-all.sh

```



### 5.3 HBase 安装 && 配置



## 




## 6. 集群运行

```shell
/usr/lib/java-1.7.0/jdk1.7.0_80
#为了测试集群配置,可先关闭防火墙
#systemctl stop firewalld.service

hdfs namenode -format <cluster-name>

#ALL HDFS daemons could be started by using $HADOOP_PREFIX/sbin/start-dfs.sh
#you must have write permissions on ${HADOOP_PREFIX}/logs on every node
hadoop-daemon.sh --config $HADOOP_CONF_DIR --script hdfs start namenode
hadoop-daemons.sh --config $HADOOP_CONF_DIR --script hdfs start datanode

#ALL YARN daemons could be started by invoking $HADOOP_PREFIX/sbin/start-yarn.sh
yarn-daemon.sh --config $HADOOP_CONF_DIR start resourcemanager
yarn-daemons.sh --config $HADOOP_CONF_DIR start nodemanager
yarn-daemon.sh --config $HADOOP_CONF_DIR start proxyserver


mr-jobhistory-daemon.sh --config $HADOOP_CONF_DIR start historyserver
```

```shell
#Namenode memory configuration
HADOOP_NAMENODE_OPTS
HADOOP_SECONDARYNAMENODE_OPTS

#default cluster log directory, set in hadoop-env.sh
HADOOP_LOG_DIR

```


```shell
#配置${HADOOP_PREFIX}/etc/hadoop/slaves,每一行添加一个data node

#${HADOOP_PREFIX}/etc/hadoop/log4j.properties配置

```








###　常用参数说明

#### core-site.xml

	fs.defaultFS : hdfs URI

	io.file.buffer.size : Size of read/write buffer used in SequenceFiles,4K is the default value, which is small.

#### hdfs-site.xml

	dfs.namenode.name.dir

	dfs.replication

	dfs.blocksize

	dfs.hosts / dfs.hosts.exclude

	dfs.blocksize : 128MB default

	dfs.namenode.handler.count : More NameNode server threads to handle RPCs from large number of DataNodes.

	dfs.datanode.data.dir

	dfs.datanode.du.reserved : reserved space for non-hdfs use, unit is bytes.


#### yarn-site.xml

>ResourceManager Config

	yarn.admin.acl :　who can be admin of the YARN cluster

	yarn.log-aggregation-enable

	yarn.resourcemanager.address : for clients to submit jobs

	yarn.resourcemanager.scheduler.address : for ApplicationMasters to talk to Scheduler to obtain resources.

	yarn.resourcemanager.resource-tracker.address : for NodeManagers.

	yarn.resourcemanager.admin.address :  for administrative commands.

	yarn.resourcemanager.webapp.address :　web-ui

	yarn.resourcemanager.hostname　: could be overrided by above parameters

	yarn.resourcemanager.scheduler.class : CapacityScheduler (recommended), FairScheduler (also recommended), or FifoScheduler

	yarn.scheduler.minimum-allocation-mb : Minimum limit of memory to allocate to each container request at the Resource Manager.

	yarn.scheduler.maximum-allocation-mb : Maximum limit of memory to allocate to each container request at the Resource Manager.

	yarn.resourcemanager.nodes.include-path : If necessary, use these files to control the list of allowable NodeManagers.

	yarn.resourcemanager.nodes.exclude-path


>NodeManager Config

	yarn.nodemanager.linux-container-executor

	yarn.nodemanager.resource.memory-mb : Defines total available resources on the NodeManager to be made available to running containers

	yarn.nodemanager.resource.cpu-vcores : should be set to the total number of cores on the machine, minus a core for each daemon process running on the machine (datanode, node manager, and any other long-running processes).

	yarn.nodemanager.vmem-pmem-ratio : Maximum ratio by which virtual memory usage of tasks may exceed physical memory

	yarn.nodemanager.local-dirs : Comma-separated list of paths on the local filesystem where intermediate data is written,Multiple paths help spread disk i/o.

	yarn.nodemanager.delete.debug-delay-sec : Number of seconds after an application finishes before the nodemanager's DeletionService will delete the application's localized file directory and log directory.

	yarn.nodemanager.log-dirs : where logs are written,Multiple paths help spread disk i/o.

	yarn.nodemanager.log.retain-seconds : default 10800, retain log files on the NodeManager Only applicable if log-aggregation is disabled.

	yarn.nodemanager.remote-app-log-dir : default /logs, HDFS directory where the application logs are moved on application completion. Need to set appropriate permissions. Only applicable if log-aggregation is enabled.

	yarn.nodemanager.remote-app-log-dir-suffix : Only applicable if log-aggregation is enabled.

	yarn.nodemanager.aux-services : default mapreduce_shuffle, Shuffle service that needs to be set for Map Reduce applications.

	yarn.log-aggregation.retain-seconds : How long to keep aggregation logs before deleting them. -1 disables. Be careful, set this too small and you will spam the name node.

	yarn.log-aggregation.retain-check-interval-seconds

	yarn.nodemanager.resource.cpu-vcores : should be set to the total number of cores on the machine, minus a core for each daemon process running on the machine (datanode, node manager, and any other long-running processes).


>Node Health Config

	yarn.nodemanager.health-checker.script.path : Script to check for node’s health status.

	yarn.nodemanager.health-checker.script.opts

	yarn.nodemanager.health-checker.script.interval-ms

	yarn.nodemanager.health-checker.script.timeout-ms

	yarn.nodemanager.disk-health-checker.min-healthy-disks

#### mapred-site.xml

>MapReduce Applications Config

	mapreduce.framework.name : yarn

	mapreduce.map.memory.mb : resource limit for maps, partially determines the container size.

	mapreduce.map.java.opts : child jvm parameters

	mapreduce.reduce.memory.mb : resource limit for reduces, together with mapreduce.map.memory.mb determine the container size.

	mapreduce.reduce.java.opts :　heap-size for child jvms of reduces.

	mapreduce.map.cpu.vcores : cpu cores allocated for container

	mapreduce.reduce.cpu.vcores : cpu cores allocated for container

	mapreduce.task.io.sort.mb　: memory-limit while sorting data for efficiency.

	mapreduce.task.io.sort.factor : streams merged at once while sorting files.

	mapreduce.reduce.shuffle.parallelcopies : number of parallel copies run by reduces to fetch outputs from very large number of maps.


>MapReduce JobHistory Server Config

	mapreduce.jobhistory.address

	mapreduce.jobhistory.webapp.address

	mapreduce.jobhistory.intermediate-done-dir : Directory where history files are written by MapReduce jobs.

	mapreduce.jobhistory.done-dir : Directory where history files are managed by the MR JobHistory Server.




### Spark Configuration

Driver schedules tasks on the cluster, it should be run close to the worker nodes,preferably on the same local area network.If you’d like to send requests to the cluster remotely, it’s better to open an RPC to the driver and have it submit operations from nearby than to run a driver far away from the worker nodes.

There are two deploy modes that can be used to launch Spark applications on YARN. In yarn-cluster mode, the Spark driver runs inside an application master process which is managed by YARN on the cluster, and the client can go away after initiating the application. In yarn-client mode, the driver runs in the client process, and the application master is only used for requesting resources from YARN.

--master parameter is yarn-client or yarn-cluster

Set HADOOP\_CONF\_DIR or YARN\_CONF_DIR which contains client side configuration.These configs are used to write to HDFS and connect to the YARN ResourceManager.

```shell
# modify SPARK_DIST_CLASSPATH to include hadoop package jars
# add an entry in conf/spark-env.sh

### in conf/spark-env.sh ###

# either
# If 'hadoop' binary is on your PATH
export SPARK_DIST_CLASSPATH=$(hadoop classpath)

# or
# With explicit path to 'hadoop' binary
export SPARK_DIST_CLASSPATH=$(/path/to/hadoop/bin/hadoop classpath)

# or
# Passing a Hadoop configuration directory
export SPARK_DIST_CLASSPATH=$(hadoop --config /path/to/configs classpath)

```

spark-submit --class org.desmond.spark.minitest.MiniSpark1 ./target/spark-test.jar

### Parquet

```shell
<dependency>
	<groupId>org.apache.parquet</groupId>
    <artifactId>parquet-avro</artifactId>
    <version>1.9.0</version>
</dependency>


```


### ORC


#### Index

	file level : statistics about the values in each column across the entire file

	stripe level : statistics about the values in each column for each stripe

	row level : statistics about the values in each column for each set of 10,000 rows within a stripe



	The file and stripe level column statistics are in the file footer so that they are easy to access to determine if the rest of the file needs to be read at all

	Row level indexes include both the column statistics for each row group and the position for seeking to the start of the row group.

#### Table properties

	orc.compress				ZLIB			{NONE, ZLIB, SNAPPY}
	orc.compress.size		  262,144			compression chunk size
	orc.stripe.size			67,108,864		memory buffer in bytes for writing
	orc.row.index.stride		10,000			number of rows between index entries
	orc.create.index			true			create indexes?
	orc.bloom.filter.columns	””				comma separated list of column names
	orc.bloom.filter.fpp		0.05			bloom filter false positive rate

~


### Daemon(admin)

-	$HADOOP_CLASSPATH is NOT for user specific dependencies.

```shell
$HADOOP_PID_DIR
$HADOOP_LOG_DIR
$HADOOP_HEAPSIZE / $YARN_HEAPSIZE

$HADOOP_PREFIX
```

### Job
### Individual operation

```shell
# ${HADOOP_HOME}/etc/hadoop/slaves 配置data node节点,每行一个


```
