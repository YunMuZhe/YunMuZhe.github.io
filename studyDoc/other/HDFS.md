# 1. HDFS - Hadoop Distribute File System

> 分布式文件系统的好处：将文件分散储存在不同节点上，读取时可以并发的去读取各个节点的子文件

设计一个分布式文件系统，需要考虑哪些问题？
1. 要分到几个节点上
2. 每份文件创建几个备份以应对硬件故障风险

1. 组成的主机变多了，任务分配问题
2. 读写速度问题
3. 数据安全性的问题

> 故障检测：某个节点宕机能够立即了解到
> 
> 实现：心跳包。  
> 从节点定时向主节点发送一条很小的数据，主节点有一个接受的函数，如果有从节点没有发送则可以检测出是否宕机

如果设置文件副本为3，有一个节点宕机之后主机发现文件副本<3,则从另外两个节点拿出来然后再去另一个节点储存一份备份以满足备份数量为3.

HDFS只能一次性写入，不支持修改，只能创建一个新的文件，上传覆盖--可以保证数据一致性

不是把文件取出来再进行计算而是**把计算逻辑发送到节点上然后在节点上进行计算将结果返回**

不适合的场景：
1. 实时访问     **一般和HDFS有关的应用都是离线分析**
2. 少量数据的存储

# 2. Block

1. 大小默认为128MB，所以不适合存储小文件（小于128MB的），因为就算文件为12K，依然占用128MB，如何解决？ **--将小文件拼成大文件再上传**
2. block大了，寻址开销减小，耗时降低
3. block大了，若单个文件太小，就会浪费硬盘空间


# 3. NameNode和DataNode

用户要上传文件，发送请求给NameNode
NameNode进行如下工作（相当于一个管家，负责管理文件如何切块）：

1. 计算文件应该切几块
2. 计算应该放在哪几个子节点上（容量和资源（包括带宽、硬盘） ）
3. 记录文件和Block的关系及Block和节点之间的关系
4. 将存储策略响应发送给用户
5. 用户接受响应后与DataNode通信->将文件分块并一块一块的发送到DataNode,不是并发的写到所有DataNode上而是发送到一个点再由它复制过去。

DataNode进行如下工作（只面向Block，不管文件）：

1. 用户将Block发送到DataNode上后DataNode存好之后通过内部局域网根据策略发送到其他节点
2. 不同的Block是并发写的，而每一个Block的备份文件的写是由指定的DataNode去完成的
2. 所有节点写成功后返回用户
3. 用户告诉NameNode存储完毕 

**DataNode定时向NameNode发送已存储的Block，让NameNode及时掌握Block分布真实数据，可以保证备份数据的数量正常。**

用户要下载文件，发送请求给NameNode
NameNode进行如下工作：

1. 查询文件和Block关系记录，再查Block和节点之间的关系的记录，返回block location
2. 将空闲DataNode发送给用户告诉他可以去那里下载

DataNode进行如下工作：

1. 用户拿着Block location并发的去各个节点拿数据
2. 告诉NameNode已经读完数据


启动的时候DataNode自检，然后将当前状态发送到NameNode，NameNode更新快照（文件与Block的关系、Block与DataNode的关系）。
如果自检的时候有DataNode发生故障没有发送自己状态，导致NameNode中有的File的Block一个都没有，那么HDFS会进入一个安全模式，拒绝用户访问。

有快照，但是DataNode里面没有任何东西，这时候自检就过不去，如果那个文件没用了，可以通过手动命令退出安全模式。

# 4. SecondaryNameNode（CheckPointNode） -- 快照的管理和定时备份

主要进行系统的监控，为后台进程，定期保存HDFS元数据的快照，降低宕机带来的损失。
快照的管理者为SecondaryNameNode

# 5.CheckPoint

1. fsimage是总文件 -- 不是实时记录新数据，而是某个时刻之前的文件系统的快照
2. edits是临时文件 -- 记录某个时刻之后的快照
3. 每到一个CheckPoint，NameNode都会将edits和fsimage发送到SecondaryNameNode，而NameNode开始往edits.new中记录新的内容。SecondaryNameNode会将edits和fsimage进行整合，整合为一个fsimage.chpt，然后将fsimage.chpt发送回NameNode替换掉之前的fsimage，NameNode用edits.new直接替换掉edits


CheckPointNode中有fsimage的备份，所以如果NameNode宕机，可以通过CheckPointNode恢复某个CheckPoint的状态。


Hadoop2
主要解决Hadoop1的单点故障问题：HDFS支持NameNode联邦与高可用（HA：在挂掉的一瞬间有替补能够立即顶上去）
HA：部署一对主/备NameNode，都需要访问到edits文件，ZooKeeper管理NameNode之间的切换


上传 ： rz
下载 ： sz /src

# 6. 面向对象编程：
1. 类： 对现实世界中同一类事物的共同的特征的抽象
2. 对象：是类的实例，与现实世界中一个具体的事物相对应

# 7. classPath
1. JavaSE: /bin
2. JavaEE: /WIN-INF/classes
3. 代表保存了.class文件的文件夹，ClassLoader会从classPath下去加载.class文件
4. 项目关联的配置文件，如果放到sec目录下，Eclipse会自动将配置文件拷贝到classPath下
5. 如果配置文件也放到src下，逻辑上有些混乱，所以Eclipse提供了一种文件夹--sourceFolder，所有该文件夹下的内容，也会被自动拷贝到classPath下。

# 8. HDFS文件操作上传的权限问题
报错：
org.apache.hadoop.security.AccessControlException: Permission denied: user=Administrator, access=WRITE, inode="/":root:supergroup:drwxr-xr-x
1. 学习阶段，Hadoop上并没有创建用户和用户组，默认root组，supergroup组
2. 本机使用HDFS API时，默认使用当前Windows登录的用户来访问HDFS
3. 在学习阶段可以关闭HDFS的权限认证













·