# MapReduce、HBase
## 1.关于MapReduce运行时间的问题
1. MapReduce在小数据量与MySQL相比并没有那么快，设计决定它的运行不是实时的，至少要运行一会才行，所以小数据量并不会有很高的效率。
2. MapReduce耗时曲线与数据量保持一个线性的增长，而MySQL所处理的数据当达到一个数量级之后，耗时会呈指数级增长。

## 2.无限大表数据库HBase
1. 特点：
	1. 大，一个表可以包含上亿行，上百万列
	2. 面向列,传统关系型数据库面向行（与生活中的数据一样，一条一条的，并且也是以行来存储的），在HBase中，看起来数据也是一行一行的，但是存储却是面向列的。
	> 平常进行数据分析大部分都是面向列的，比如对英语成绩进行分析，就需要把一行一行的数据取出来，而HDFS存储的数据可能这一部分在slave1上，另一部分在slave2上，就需要跨节点传输数据。而按列存储就可以在某些slave上进行分析，而不用跨节点传输数据，这样就避免了自愿的浪费。
	3. 稀疏：对于值为NULL的数据，不占用存储空间
	4. 针对于每个cell（每个具体的字段）保存多个版本，由时间戳标记。
解决浪费存储空间的策略：设置保存的时间范围（如：7天）、设置保存的个数（如：10个）
	5. 可以实现大数据的实时查询，所以有想要实时查询的大数据，一般都放到HBase中
	6. 存数据时，会自动对RowKey排序，排序的逻辑：字典顺序


2. 组成
	1. HBase表中存在一个默认的列族RowKey（行键），是一行数据的唯一标识，查询的唯一标识，默认存在
	2. HBase表中的列不叫列，叫列族（Column Family），比如声明两个列族：info、score。
	3. 在每一个列族下面可以声明具体的列，比如在info列族下声明列：name、age
	4. 声明表时只需要声明列族，添加数据的时候再声明列族下面的具体列，可以动态添加列，一行数据的某列没有数据是允许的，因为并不占用空间
	5. 理解：真实存数据时使用的是列族，而下面的列是虚拟的，列族中的列存储数据时是这样的：name：Jack，age：12（Info列族下有name和age列），Jack前面的name称为限定符
	6. 所有的数据公用的只有列族，而列仅仅只是那一行数据自用的

3. HBase Shell
	1. 命令大小写敏感，命令结束不需要加分号
	2. 常用命令
		1. `list`--列出表
		2. 删掉表：先`disable 'table_name'`使此表不运行，然后才能`drop 'table_name'`删除表
		3. 建表：`create  'student','name','profile','score'`
			1. 解释：student为表名
			2. 后面的name、profile、score均为列族
			3. RowKey不需要声明，默认即存在
			4. 创建有name、profile、score列族的student表
		4. 插入数据： `put 'student','11','name:name','John'`
			1. student为表名
			2. 11为其RowKey
			3.  name：name  -- 列族、限定符
			4.  John是具体的名字
			5.  其实是插入的一行数据的一个单元格的数据
			6.  插入之后，结果为name列族下加了一个name的列，name这个列里面加了一个John的真实数据
		5. 查看表中数据：`scan 'student'`
		6. 查看表结构： `desc 'student'`

4. HBase的API
	1. 在HBase中，涉及查询时，查询到的数据应该封装成DO对象
	2. 涉及的问题是：HBase表中的字段可能不断增加
	3. 有如下两种方法：
		1. 和以往一样，当更改时对应的也更改属性集
		2. 声明一个针对列族的集合
			1. 如private List<Map> name;
			2. name.add(new Map("name", "Tom"));
			3. 这样的话设计相对复杂，而且也不能用Get了
	4. HBase底层存储数据时，都是字节数组。拿出数据来需要自己封装.
	5. 如：`student.setId(Integer.parseInt(Bytes.toString(result.getRow())));`
	  `Bytes.toInt();`也可以，不过容易出现异常，最为稳妥的是先转为String然后再转为Int
	6. `result.getValue(family, qualifier); `指定列族和列名才能取出数据.
	
	如：`result.getValue(Bytes.toBytes("name"), Bytes.toBytes("name"))`，
	
	需要注意的是可能返回null。

	7. `scan.setStartRow(Bytes.toBytes(startRK)); //从哪一个RowKey开始，包括起始`
   `scan.setStopRow(Bytes.toBytes(endRK));//到哪一个RowKey结束，不包括结束`
	
 	包前不包后
	8.  `byte[] alias=result.getValue(Bytes.toBytes(FAMILYS[0]), Bytes.toBytes("alias"));`
		
		`if(alias!=null){ //规避空指针异常--NPE--NullPointerException
		student.setAlias(Bytes.toString(alias));
		}`


## 3.常识性知识

1. 企业开发中，对于JavaBean一般只用包裹类，如Integer、Double而不是int、double，虽然装箱拆装会有部分资源消耗，但是有一个很大的好处：包裹类可以接受NULL值
2. 程序员水平提升途径：
	1. 先写注释，将思路写出来，然后再写代码就简单了
	2. 应用反射等设计模式
3. 一个架构师负责的是一个软件的整体架构，好的架构是好的设计模式的堆叠，对设计模式的学习是很漫长的过程，分为三个过程：
	1. 概念的学习 -- 大学阶段
	2. 研究组件的原理中去识别出设计模式，如SSM，并且可以懂得为什么要用这种设计模式，工作中一定要多研究
	3. 在具体问题中去应用模式
4. 应用设计模式会将软件的迭代、修改变得非常简单
5. 对于那些只用执行一次的代码，如果是静态的，就可以放到静态代码块里，而如果是实例的，放到构造函数里即可
6. 项目解耦的基本原则之一：不在层与层之间传递某个层特有的对象，这会人为地造成耦合
	
例如：Web层：Servlet、Filter、Listener、Request、Response、ServletContext
		 DAO层：Connection、ResultSet、Statement、事务管理的代码
		 根据业务逻辑，实务操作应该在Service层进行，但是Service层不应该出现连接对象，为了解耦，会通过特殊工具类：TransactionManager封装所有实务操作的代码，实现Service和DAO在事务管理上的解耦



> ******需补充
> ## 4. Hive和HBASE相连
> 	1. 方法
> 	2. 开发者使用数据方式
> 		1. HiveQL传入Hive，然后由HiveQL去进行翻译，在HBase中查询
> 		2. HBase API命令直接去HBase中查找数据


## 复习
1. Hive
	1. Hadoop平台的数据仓库工具
	2. 由Facebook开发，后贡献给Apache基金会
	3. 可以将操作者的HiveQL（类SQL）转化成分布式计算任务，在Hadoop集群上执行
	4. Hive1.x主要支持将HiveQL转化成MapReduce任务，Hive2.x主要支持将HiveQL转换成Spark或Tez任务
	5. 安装和配置
	6. API和JDBC完全一样，换个配置即可

2.Hadoop的组件
	1. HDFS：在所有节点上都有运行，NameNode和DataNode
	2. Yarn：ResourceManager、NodeManager、ApplicationManager、Container，进行资源管理
	3. Hive对数据操作，走HDFS，而进行操作时，走计算引擎（MR、Spark、Tez），然后计算引擎再转至Yarn那。