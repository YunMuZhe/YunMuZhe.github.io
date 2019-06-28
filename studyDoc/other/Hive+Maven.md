# 1.新课--Hive--数据仓库
Hive本身不存储真实数据，只存储元数据

真实数据存储在HDFS上

适用于熟练掌握SQL语言的数据编程人员
提供一套类SQL语言--HiveQL，并且可以转换为MapReduce操作执行

没有分布式计算的功能，借助于分布式计算引擎，如Spark、MapReduce

HiveThriftServer--接收用户请求，支持面很广，支持各种客户端，Hive命令、JDBC代码、微软的ODBC...均可接受

HiveDriver（核心）--收到用户请求后发到这里，转化为操作，包括解析器、解释器、编译器、执行器。
将HiveQL变为MapReduce任务是它的主要功能。单纯的操作直接去查HDFS。

Metastore--元数据存储，也存在与一个数据库中，默认是Apache的一个数据库，但是这个数据库不支持并发访问，所以商业上一般会换为其他数据库，如Mysql、MariaDB

# 2.常识
XML第一行必须为XML的声明，不可为其他的东西

# 3. Hive
1. 类SQL语言
	1. 支持插入、支持create table as select，集成MapReduce脚本
	2. 不支持更新操作和事务（HDFS的数据不支持更新数据）
	3. 子查询和联接操作不是很支持
2. 数据类型
	1. 存在BOOLEAN
	2. 字符型只有一种--String，类似于varchar，而没有char和varchar
3. 复杂类型
	1. Array--一组有序字段，字段类型必须相同
	2. Map--无序键值对
	3. STRUCT--一组字段，字段类型可以不同 
4. 大数据的数据导入：

> 	1. 来自于文件
> 	2. 文件中每行是一条数据，一条数据的多个字段由固定的分隔符，如空格、\t、','
> 	3. 按照分隔符自动分割然后发送到Hive中的表
> 	4. 在建表时必须制定表中数据在现实文件中所使用的分隔符
> 	5. Hive本身不存储数据，它会将数据存储至HDFS中


5. 托管表
 数据文件存放在Hive数据仓库中，由Hive平台管理，删除表数据（文件）也删
6. 外部表
	1. 数据文件指定存放位置，删除表数据（文件）不删
	2. 建表时如果用LOCATION子句关联一个HDFS上的文件，那么该文件不会被拷贝到HDFS上的Hive仓库中，删除该Hive表时，HDFS上的数据不会被删掉
	3. 如果是后期使用LOAD命令向外部表中导入数据，这部分数据会被保存到Hive的仓库中，删除外部表，这部分数据会被删除。

7. Load
	1. load data local inpath '' overwrite into table tablename;
	2. local指的是操作系统的文件路径，否则默认为HDFS的文件路径
	3. 有overwrite代表是否将要导入的表中的数据删除
	
# 4. Hive的UDF和UDTF
1. hive的用户定义函数

	当HiveQL无法满足复杂的数据清洗或分析操作时，用户可以以Java代码的形式提供数据的处理逻辑，并注册为一个自定义函数，嵌入到HiveQL中执行。



# 5. 业务需求
1. 对619一天的StationStatus数据进行处理，将原始数据变成关系型数据
2. 基于Hive对数据进行基本的分析
3. 分析后的数据存入HBase中
4. 构建基于JavaWeb+HBase的可视化应用

# 6. 今日操作
在HDFS上创建对应的目录

	hdfs dfs -mkdir /nybike
	hdfs dfs -mkdir /nybike/test
	rz -y  //将nybike_status_sample.txt通过XSHELL上传到虚拟机上
	//将nybike_status_sample.txt文件上传到HDFS上：
	hdfs dfs -put /opt/station_status_sample.txt /nybike/test 

# 7. 数据的时间问题
所有站点数据统一用整条数据的时间戳，会造成冗余，但是可以降低处理难度
把时间处理成YYYYMMDDHHmm格式的纽约市时间然后再生成（时间：sid）的格式存入HBase中
如此设计字段格式的原因：


> # 8. HBase -- Hadoop Database
> 无限大表数据库
> 
> 一张表中可以保存上亿行，上百万列的数据
> 
> HBase中的数据查询效率非常高，但是有一个限制，**默认仅支持使用> RowKey(自己设置，仅一个字段)进行查询**
> 
> 此RowKey应该是我们查询经常用到的数据，在NYBike项目中，查基本数据，一般用到两个字段：时间、sid，所以我们设计为   (时间：sid)这样一个字段

# 9. 接下来的流程
将day08打包为jar包，上传到master /opt上面，在hive中通过命令添加这个jar包：`add jar /opt/udtf.jar` 声明一个临时函数

在HiveQL中使用该函数

# 10. Maven操作
![run as基本操作解释](https://upload-images.jianshu.io/upload_images/6324621-eaa61f11e02ca0b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/6324621-5c13b609dfeec042.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/6324621-e0274954e6fb53ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



除此之外，还出现了Jar包缺失的错误：
![Maven报错解决.png](https://upload-images.jianshu.io/upload_images/6324621-be9301b9de17728e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
附：
[[ERROR] No compiler is provided in this environment. Perhaps you are running on a JRE rather than a JDK?错误解决方案](https://blog.csdn.net/lslk9898/article/details/73836745)


# 复习
1. Hadoop
	1. 来自于Apache，开源，占有率最高的大数据平台，（CDH）
	2. HDFS：
	3. MapReduce：
	4. Hadoop生态系统的其他组件
		1. Hive
		2. HBase
		3. Pig
		4. Zookeeper
		5. Sqoop
		6. Spark
		7. 
2. HDFS
	1. 核心进程--作用、运行在哪个节点上
		1. NameNode
		2. DataNode
		3. SecondaryNameNode
	2. HDFS的读流程（下载） 和写流程（上传） -- 掌握
	3. Block的概念
	4. HDFS的命令 和 HDFS的Java API
3. MapReduce
	1. MapReduce的编程模型
		1. map（）：在所有数据上进行并行计算的逻辑
		2. reduce（）： 对map函数的结果进行汇总
	2. MapReduce的API
		1. Map
		2. Reducer
		3. Driver
		4. 打包上传运行/Eclipse远程调试
	3. MapReduce任务的执行过程：先发到从节点运行Map，再从本地排序，再发送到运行Reduce的节点....
	4. Yarn的架构和运行逻辑 -- 需要了解
		1. ResourceManagement
		2. NodeManager
		3. ApplicationManager
		4. Container
		