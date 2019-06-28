# Hive2.3.5安装文档
## Centos7安装Mariadb（如已安装，可跳过）
### 1. 上传离线安装包(mariadb.tar.gz)

	rz -y 

### 2. 解压缩

	tar zxvf /opt/mariadb.tar.gz -C /opt

### 3. 安装

	yum install /opt/mariadb/*

### 4. 设置MariaDB

	启动MariaDB
	systemctl start mariadb
	
	设置开机启动
	systemctl enable mariadb

	MariaDB初始化配置
	mysql_secure_installation

	密码验证，初次直接敲回车
	Enter current password for root (enter for none):<–初次运行直接回车
	
	设置root用户密码-教学这里使用root
	Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车
	New password: <– 设置root用户的密码
	Re-enter new password: <– 再输入一次你设置的密码

	其他配置：
	Remove anonymous users? [Y/n] <– 是否删除匿名用户，回车
	Disallow root login remotely? [Y/n] <–是否禁止root远程登录,回车,
	Remove test database and access to it? [Y/n] <– 是否删除test数据库，回车
	Reload privilege tables now? [Y/n] <– 是否重新加载权限表，回车

	登录MariaDB:
	mysql -uroot -proot

	


## Hive2.3.5安装
### 1. 主节点获取Hive安装包

	方式一：从Windows主机上传(xshell)

		rz -y

	方式二：主节点直接从网上下载安装包
	
		cd /opt
		wget https://mirrors.tuna.tsinghua.edu.cn/apache/hive/hive-2.3.5/apache-hive-2.3.5-bin.tar.gz

### 2. 解压缩

	tar zxvf /opt/apache-hive-2.3.5-bin.tar.gz -C /opt/

	修改文件夹名称
	mv /opt/apache-hive-2.3.5-bin /opt/hive-2.3.5

### 3. 配置环境变量

	编辑文件
	vim /etc/profile
	添加
	export HIVE_HOME=/opt/hive-2.3.5
	export PATH=$HIVE_HOME/bin:$HIVE_HOME/conf:

	让配置生效
	source /etc/profile

### 4. 配置Hive

1. 配置hive-env.sh

		复制模板
		cd /opt/hive-2.3.5/conf
		cp hive-env.sh.template hive-env.sh
	
		编辑文件	
		vim /opt/hive-2.3.5/conf/hive-env.sh
	
		在最末尾添加
		export HADOOP_HOME=/opt/hadoop-2.6.5
		export HIVE_HOME=/opt/hive-2.3.5
		export HIVE_CONF_DIR=/opt/hive-2.3.5/conf
		export JAVA_HOME=/opt/jdk1.8.0_161
		export HIVE_AUX_JARS_PATH=/opt/hive-2.3.5/lib

2. 配置hive-site.xml

		编辑文件
		vim hive-site.xml
	
		<?xml version="1.0" encoding="UTF-8" standalone="no"?>
		<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
		<configuration>
			<property>
				<name>javax.jdo.option.ConnectionURL</name>									修改Hive的元数据库为MariaDB
				<value>jdbc:mysql://master:3306/hive?createDatabaseIfNotExist=true</value>
			</property>
			<property>
				<name>javax.jdo.option.ConnectionDriverName</name>                          连接数据库的驱动
				<value>com.mysql.cj.jdbc.Driver</value>
			</property>
			<property>
				<name>javax.jdo.option.ConnectionUserName</name>                            Hive访问MySQL时使用的用户名
				<value>hadoop</value>
			</property>
			<property>
				<name>javax.jdo.option.ConnectionPassword</name>							Hive访问MySQL时使用的密码
				<value>hivepwd</value>
			</property>
			<property>
				<name>hive.metastore.warehouse.dir</name>									在HDFS中Hive的仓库所在文件夹
				<value>hdfs://master:8020/hive/warehouse</value>
			</property>
			<property>
				<name>hive.exec.local.scratchdir</name>
				<value>/opt/hive/exec</value>
			</property>
			<property>
				<name>hive.downloaded.resources.dir</name>
				<value>/hive/downloadedsource</value>
			</property>
			<property>
				<name>hive.querylog.location</name>
				<value>/hive/logs</value>
			</property> 
		</configuration>

3. 配置Hive的日志目录    log4j是一个日志管理第三方工具

		拷贝文件
		cp hive-log4j2.properties.template hive-log4j2.properties
	
		编辑文件
		vim hive-log4j2.properties
	
		property.hive.log.dir = /opt/hive/log

4. 拷贝Mysql连接jar包到lib目录下

		cp /opt

		上传jar包
		rz -y
		
		cp /opt/mysql-connector-java-8.0.13.jar /opt/hive-2.3.5/lib/

5. 解决SLF4J重复问题  Hadoop本身也带了，运行时会有冲突，有异常

		rm /opt/hive-2.3.5/lib/log4j-slf4j-impl-2.6.2.jar

### 5. Hive元数据库配置

	进入MariaDB
	mysql -uroot -proot

	建库
	create database hive;

	配置权限
	grant all on hive.* to hadoop@'master' identified by 'hivepwd';

	使配置生效
	flush privileges;



### 6. 启动hive并测试

	首次启动前，初始化hive元数据库
	schematool -dbType mysql -initSchema

	启动hive
	hive

	测试Hive
	create database testdb;
	use testdb;
	create table student(id int,name string);
	insert into student values(1,'Tom');
	select * from student;