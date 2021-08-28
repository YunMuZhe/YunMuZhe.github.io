# 索引

## 用途

提升访问效率

## 存储位置

索引和实际的数据都是存储在磁盘中的，只不过在进行数据读取的时候会优先把索引加载到内存中。

MySQL数据库中，什么情况下设置了索引但无法使用

1. 没有符合最左前缀原则
2. 字段进行了隐私数据类型转化
3. 走索引没有全表扫描效率高

存储的数据是K-V格式的数据。K为要查询的字段，V为存储在哪一个文件中及其offset

OLAT：联机分析处理——数据仓库——HIVE，不要求在很短的时间返回结果，只需要

OLTP：联机事务处理——数据库——MySQL、db2、Oracle——业务系统的需要，在很短的时间返回结果

#### ---------------数据结构为B+树，为什么呢？

#### 没有选择哈希表的原因：

1. 哈希表是为了将数据进行散列的数据结构，如果hash算法设计不好的话，很容易造成哈希碰撞、哈希冲突，会造成存储空间的浪费，查询效率低
2. 当进行范围查找的时候，需要挨个进行对比，效率极低

hash类型的索引：查询单条快，范围查询慢

btree类型的索引：b+树，层数越多，数据量指数级增长，范围查找效率高

B树特点：多叉树、有序、平衡

阶：每个节点最多存储多少个数据

B树每个节点存储的数据有：键值即表中记录的主键、指针即存储子节点地址信息、数据即表记录中除主键外的数据

B+树相比于B树而言，将所有数据全部放到了叶子结点，而非叶节点中只放置了key值和子节点地址信息

#### MySQL索引的B+树一般有几层？

一般情况下，3到4层的B+树足以支撑千万级的数据量存储

key值在存储容量上起到了决定性的作用，当key值占用空间较大时，每个磁盘块中能放的节点数就会减少，会导致同样层数，同样节点数的B+树存储的索引节点变少，减少存储量。

#### 一般选择索引的时候用int类型还是varchar类型？

此处需要查看key值所占用的存储空间，int占用4个字节，因此如果varchar的长度小于等于3时使用varchar，若大于4则选择int

#### 选择索引时，int类型要不要自增呢？

在满足业务场景的情况下，能自增最好自增。因为新增数据时如果只在最右侧新增数据，则可以减少索引的维护成本。

#### MySQL的一张表中可以包含几个索引？

理论上来说没有限制，但是并不是越多越好，不是给每一个列添加索引后都会加快查询效率。

#### 在刚刚的图中，发现叶子结点存储索引，如果一个表有3个索引，那么有几棵B+树？

3棵B+树，一个索引对应一棵树。

#### 如果有三棵树的话，数据存储几份？

数据只存储一份，不会冗余存储数据

#### 如果只有一份的话，那么索引的叶子结点中存储什么？

innodb存储引擎在进行数据插入的时候，必须要将数据跟某一个索引列绑定存储，如果表中有主键则用主键，没有主键用唯一键，没有唯一键会自动生成一个6字节的rowid来进行数据存储，也就是说，数据跟某一个索引列放在一起，所以索引树的叶子结点中存储的是跟数据绑定的索引列的key值。

id,name,age,gender id主键、name普通索引

#### 回表

select * from table where name = 'zhangsan';

先根据name的值查询nameB+树，找到对应的id，再根据id区idB+树上查找所有的数据，这个过程就称之为回表

回表的效率较低，不推荐

#### 索引覆盖

select id, name from table where name = 'zhangsan';

根据name区nameB+树上找到叶子结点，能够取到name、id的值，不需要回表了，这个过程叫做索引覆盖

查询的索引树的叶子结点中包含了所有要查询的字段，推荐。

#### 最左匹配

id, name, age, gender id主键 name,age组合索引

select * from table where name = 1 and age = 1; 能匹配到索引

select * from table where name = 1; 能匹配到索引

select * from table where age = 1; 不能匹配到索引

select * from table where age = 1 and name = 1; 能匹配到索引 优化器

#### 索引下推

select * from table where name = 1 and age = 1;

没有索引下推之前：先根据name去存储引擎之中拉取数据，然后在server层对age做数据过滤

有了索引下推之后：直接根据name和age的值区存储引擎拉取数据，不需要在server层做数据过滤

除此之外，or关键字是否使用索引需要分情况讨论，where name =1 or name = 2; 除此之外还有using merge、use intersaction

### 索引的设计原则有哪些？

在进行索引设计的时候，应该保证索引字段占用的空间越小越好，这只是一个大的方向，还要一些细节点需要注意下：

1. 适合索引的列是出现在where条件中的列，或者连接字句中的列
2. 基数较小的表，索引效果差，没必要建立索引
3. 在选择索引列的时候，越短越好，可以指定某些列的一部分，没必要用全部字段的值
4. 不要给表中的每一个列都创建索引，并不是索引越多越好
5. 更新频繁的字段不要有索引
6. 创建索引的列不要过多，可以创建组合索引，但是组合索引的列的个数不建议太多
7. 大文本、大对象不要创建索引

### 优化SQL，考量的是你的体系性，你需要知道要考虑哪些点

一般在数据库建立的前期，我们就会对字段进行考量，比如说字段的长度、字段的类型

但这个还只是预防，大部分的优化还是上线后出现问题但主要还是在建表、有了一定数据之后再去优化的

一般出现优化问题的时候，我会从以下几个维度去考虑：

第一个还是服务器库的资源是否满足

第二,我会通过索引的判断，比如查看SQL语句的执行计划查看SQL的执行是否高效

第三，我会通过调整具体的SQL语句

第四，我会调整MySQL的参数

第五，我会进行数据库架构的调整

在我写的项目经历中，某个金融相关的SQL语句执行的就比较慢，因为它在整个执行过程中进行了大量的回表操作，所以我把我要查询的三个字段，一起组成了一个组合索引，那么在运行的时候它不需要回表了，整个效率提升了100倍都不止。

之前进行数据库备份时很慢，因此我把我的双一设置关掉了，每次执行完之后在某一个时间窗口里面直接去做的。

### 怎么处理MySQL的慢查询：

1. 开启慢查询日志，准确定位到哪个SQL语句出现了问题
2. 分析SQL语句，看看是否load了额外的数据，可能是查询了多余的行并且抛弃掉了，可能是加载了许多结果中并不需要的列，对语句进行分析以及重写
3. 分析语句的执行计划，然后获得其使用索引的情况，之后修改语句或者修改索引，使得语句可以尽可能的命中索引
4. 如果对语句的优化已经无法进行，可以考虑表中的数据量是否太大，如果是的话可以进行横向或者纵向的分表

### MySQL为什么需要主从同步？

1. **读写分离：**在复杂的业务系统中，有这么一个情景，有一句SQL语句需要锁表，导致暂时不能使用读的服务，那么就很影响运行中的业务，使用主从复制，让主库负责写，从库负责读，这样即使主库出现了锁表的情景，通过读从库也可以保证业务的正常运作。
2. **数据的热备**
3. **架构的扩展：**业务量越来越大，IO访问频率过高，单机无法满足，此时做多库的存储，降低磁盘IO访问的频率，提高单个机器的IO性能。

### MySQL复制原理是什么？

1. 当master服务器的数据改变时，master服务器将数据的改变记录到二进制binlog日志
2. slave服务器会在一定的时间间隔内对master二进制日志进行探测，查看其是否发生改变，如果发生了改变，则会开始一个IOThread和SQLThread将进入睡眠状态，等待下一次被唤醒。
3. 同时主节点为每个IO线程启动一个dump线程，用于向其发送二进制事件，并保存至从节点本地的中继日志中，从节点将启动SQL线程从中继日志中读取二进制日志，在本地重放，使得其数据和主节点的保持一致，最后IOThread和SQLThread将进入睡眠状态，等待下一次被唤醒。

也就是说：

- 从库会生成两个线程，一个IO线程，一个SQL线程
- IO线程会去请求主库的binlog，并将得到的binlog写到本地的relay-log（中继日志）文件中
- 主库会生成一个log dump线程，用来给从库的IO线程传binlog
- SQL线程，会读取relay-log文件中的日志，并解析成SQL语句逐一执行

注意：

1. master将操作语句记录到binlog中，然后授予slave远程连接的权限（master一定要开启binlog二进制日志功能；通常为了数据安全考虑，slave也开启binlog功能）
2. slave开启两个线程：IO线程和SQL线程。其中IO线程负责读取master的binlog内容到中继日志relay-log中；而SQL线程负责从relay-log中读取出binlog内容，并更新到slave的数据库中，这样就能保证slave数据和master数据保持一致了
3. MySQL复制至少需要两个MySQL的服务。当然MySQL服务可以分布在不同的服务器上，也可以在一台服务器上启动多个实例。
4. MySQL复制最好确保master和slave服务器上的MySQL版本相同(如果不能保持一致，则需要保证master主节点的版本低于slave从节点的版本)
5. master和slave节点两节点时间需同步。

![](./img/mysql-copy.png)

具体步骤：

1. 从库通过手工执行change master to 语句来连接主库，提供了连接的用户一切条件（user、password、port、ip），并且让从库知道，二进制日志的起点位置（file名 position号）； start slave
2. 从库的io线程和主库的dump线程建立连接
3. 从库根据change master to语句提供的file名和position号，IO线程向主库发起binlog的请求
4. 主库dump线程根据从库的请求，将本地binlog以events的方式发给从库IO线程
5. 从库IO线程接收binlog events，并存放到本地relay-log中，传送过来的信息会记录到master.info中
6. 从库SQL线程应用relay-log，并且把应用过的记录到relay-log.info中，默认情况下，已经应用过的relay会自动被清理purge

### MySQL聚簇索引和非聚簇索引的区别是什么？

MySQL的索引类型跟存储引擎是相关的，innodb存储引擎数据文件跟索引文件全部放在ibd文件中；而myisam的数据文件放在myd文件中，索引放在myi文件中。其实区分聚簇索引和非聚簇索引非常简单，只要判断数据跟索引是否存储在一起就可以了。

innodb存储引擎在进行数据插入的时候，数据必须要跟索引放在一起，如果有主键就使用主键，没有主键就使用唯一键，没有唯一键就使用6字节的rowid，因此跟数据绑定起来的索引就是聚簇索引，而为了避免数据冗余存储，其他索引的叶子结点中存储的都是聚簇索引的key值，因此innodb中既有聚簇索引也有非聚簇索引，而myisam中只有非聚簇索引。

### MySQL索引类型有哪些，以及对数据库的性能的影响

- 普通索引：允许被索引的数据列包含重复的值（列是普通的，不是唯一的
- 唯一索引：可以保证数据记录的唯一性
- 主键索引：是一种特殊的唯一索引，在一张表中只能定义一个主键索引，主键用于唯一标识一条记录，使用关键字primary key来创建
- 联合索引：索引可以覆盖多个数据列
- 全文索引：通过建立倒排索引，可以极大地提升检索效率，解决判断字段是否包含的问题，是目前搜索引擎使用的一种关键技术

索引可以极大地提高数据的查询速度，通过使用索引，可以在查询的过程中，使用优化隐藏器，提高系统的性能。

但是会降低插入、删除、更新表的速度，因为在执行这些写操作的时候，还要操作索引文件。

索引需要占用物理空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间，如果要建立聚簇索引，那么需要的空间就会更大，如果非聚簇索引很多，一旦聚簇索引改变，那么所有非聚簇索引都会跟着变

### MySQL索引结构有哪些，各自的优劣是什么？

![image-20210823191645548](./img/index-struct.png)

### mysql锁的类型有哪些？

基于锁的属性分类：共享锁、排它锁

基于锁的粒度分类：行级锁（innodb）、表级锁（innodb、myisam）、页级锁（innodb）、记录锁、间隙锁、临键锁

基于锁的状态分类：意向共享锁、意向排它锁

表锁：上锁时锁住整个表，下一个事务访问同一张表时，必须等待前一个事务释放了锁才能对表进行访问，特点：粒度大、加锁简单、容易冲突

行锁：上锁时锁住某一行或某些行，其他事务访问同一张表时，只有被锁住的记录不能访问，其他记录可正常访问。特点：粒度小、加锁比表锁麻烦，不容易冲突，相比表锁支持的并发要高

记录锁：记录锁也属于行锁中的一种，只不过记录锁的范围只是表中的某一条记录，记录锁是说事务在加锁后锁住的只是表的某一条记录，加了记录锁之后数据可以避免数据在查询的时候被修改的重复读问题，也避免了在修改的事务未提交前被其他事务读取的脏读问题

页锁：页级锁时MySQL中锁定粒度介于行级锁和表级锁之间的一种锁。表级锁速度快但冲突多；行级锁冲突少但速度慢。所以折中取了页级，一次锁定相邻的一组记录。特点：开销和加锁时间介于行锁和表锁之间，会出现死锁，锁定粒度介于表锁和行锁之间，并发度一般。

间隙锁：是属于行锁的一种，间隙锁是在事务加锁后锁住表记录的某一个区间，当表的相邻ID之间出现空隙时，则会形成一个区间，遵循左开右闭原则。范围查询并且查询未命中记录，查询条件必须命中索引，间隙锁只会出现在重复度的事务级别中。

临键锁：也属于行锁的一种，并且他是innodb的行锁默认算法，总结来说他就是记录锁和间隙锁的组合，临键锁会把查询出来的记录锁住，同时也会把该范围查询内的所有间隙空间也锁住，再之他会把相邻的下一个区间也锁住。

## MySQL如何做分布式锁

在MySQL中创建一张表，设置一个主键或者unique key，这个key就是要锁的key（商品ID），所以同一个key在MySQL表里只能插入一次了，这样对锁的竞争就交给了数据库，处理同一个key数据库保证了只有一个节点能插入成功，其他节点都会插入失败。

DB分布式锁的实现：

通过主键id或者唯一索引的唯一性进行加锁，说白了就是加锁的形式是向一张表中插入一条数据，该条数据的id就是一把分布式锁，例如当一次请求插入了一条id为1的数据，其他想要进行插入数据的并发请求必须等待第一次请求执行完成后删除这条id为1的数据才能继续插入，实现了分布式锁的功能。

这样lock和unlock的思路就很简单了，伪代码：

```python
def lock:
    exec sql: insert into locked-table(xxx) values(xxx)
    if result == true:
        return true
    else:
        return false
def unlock:
    exec sql: delete from lockedOrder where order_id = 'order_id'
```






































