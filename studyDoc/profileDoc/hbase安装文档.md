# HBase1.2.6安装文档
### 1. 将HBase安装包拷贝到云主机上

	cd /opt
	rz -y

### 2. 解压缩

	tar zxvf  /opt/hbase-1.2.6-bin.tar.gz -C /opt/

### 3. 配置

1. 环境变量

		编辑文件
		vim /etc/profile
       	
		添加配置
		export HBASE_HOME=/opt/hbase-1.2.6
		export PATH=$HBASE_HOME/bin:

		使配置生效
		source /etc/profile

2. 配置hbase-env.sh

		编辑文件
		vim /opt/hbase-1.2.6/conf/hbase-env.sh
		
		修改配置
		export JAVA_HOME=/opt/jdk1.8.0_161
		export HBASE_MANAGES_ZK=true

3. 配置hbase-site.xml

		编辑文件
		vim /opt/hbase-1.2.6/conf/hbase-site.xml
  
		内容如下
		<configuration>
			<property>
				<name>hbase.rootdir</name>
				<value>hdfs://master:8020/hbase</value>
			</property>
			<property>
				<name>hbase.cluster.distributed</name>
				<value>true</value>
			</property>
			<property>
				<name>hbase.master</name>
				<value>master:60000</value>
			</property>
			<property>
				<name>hbase.zookeeper.quorum</name>
				<value>master,slave1,slave2</value>
			</property>
			<property>
				<name>hbase.zookeeper.property.dataDir</name>
				<value>/home/hadoop/zoodata</value>
			</property>
		</configuration>

		配置说明
		属性1：hbase在hdfs上的目录，主机名为hdfs的namenode节点所在的主机
		属性2：指定hbase的运行模式，true代表全分布模式
		属性3：指定hbase的hmaster的主机名和端口
		属性4：指定使用zookeeper的主机地址，必须是奇数个 
		属性5：zookeeper的属性数据存储目录，如果你不想重启电脑就被清空的话就要配置这个

4. 配置regionservers

		编辑文件
		vim /opt/hbase-1.2.6/conf/regionservers

		内容如下
		slave1
		slave2 

5. 将配置好的hbase拷贝到其他节点

		scp -r /opt/hbase-1.2.6 root@slave1:/opt
		scp -r /opt/hbase-1.2.6 root@slave2:/opt

### 4. 启动HBase

	start-hbase.sh

	进入HBase的shell
	hbase shell

	访问hbase网页
	http://master:16010

	添加测试数据，测试安装正确
	
	建表：
	create  'student','name','profile','score'

	查看表结构：
	desc 'student'

	插入数据：
	put 'student','11','name:name','John'
	put 'student','11','profile:age','20'
	put 'student','11','profile:class','class1'
	put 'student','11','score:math','100'
	put 'student','11','score:lang','95'
	put 'student','20','name:name’,’Marry'
	put 'student','20','name:alias','MY'
	put 'student','20','profile:gender','fema1e'
	put 'student','20','profile:class','class2'
	put 'student','20','score:math','93'
	put 'student','20','score:eng','90'
	put 'student','32','name:name','Peter'
	put 'student','32','profile:class','class3'
	put 'student','32','score:math','87'
	put 'student','32','score:lang','85'
	
	查看表中数据
	scan  'student'

	退出shell
	quit

	关闭HBase
	stop-hbase.sh