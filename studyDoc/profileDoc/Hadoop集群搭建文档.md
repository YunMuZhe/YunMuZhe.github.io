# Hadoop集群搭建文档

### 1. 准备3台虚拟机

#### 1) 配置好主机名及IP地址

	主机名		ip地址				内存
	master		192.168.56.101		2G
	slave1		192.168.56.102		1G
	slave2		192.168.56.103		1G

	相关命令：

	1. 配置hostname： hostnamectl set-hostname master
	2. 配置ip地址： vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
	修改其中的IPADDR即可修改IP4地址
	3. 重启网卡：service network restart

#### 2)准备

关闭防火墙

	systemctl stop firewalld.service            #停止firewall
	systemctl disable firewalld.service        #禁止firewall开机启动
安装JDK

配置JDK环境变量

配置/etc/hosts文件
	
	127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
	::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
	192.168.56.101	master192.168.56.102	slave1
	192.168.56.103	slave2

### 2. 配置SSH免密登录

1. master生成密钥
   
		ssh-keygen -t rsa (四个回车)
		cd .ssh
		cp id_rsa.pub authorized_keys 
2. 将密钥发送给slave1和slave2

		ssh-copy-id root@slave1
		ssh-copy-id root@slave2
3.  测试master到slave1和slave2的ssh免密登录正常
		
		ssh root@slave1
		ssh root@slave2

#### 3. 上传hadoop安装包到/opt目录下

1. 使用xshell ssh 连接master主机
2. 进入/opt目录下
3. 使用 rz -y 命令上传文件: hadoop-2.6.5.tar.gz

#### 4. 安装Hadoop

1. 将hadoop2.6.5 解压到 /opt目录下
		
		tar zxvf /opt/hadoop-2.6.5.tar.gz -C /opt/

2. 配置环境变量

		vim /etc/profile

		export JAVA_HOME=/opt/jdk1.8.0_161
		export HADOOP_HOME=/opt/hadoop-2.6.5
		export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$JAVA_HOME/bin:$PATH:$HOME/bin

		使配置生效-仅当前shell有效-全部生效需要重启电脑
		source /etc/profile

3. 对Hadoop进行配置

	1) 修改hadoop-env.sh文件，添加jdk
	
		vim /opt/hadoop-2.6.5/etc/hadoop/hadoop-env.sh
	
		export JAVA_HOME=/opt/jdk1.8.0_161
	
	2) 修改core-site.xml
	
		vim /opt/hadoop-2.6.5/etc/hadoop/core-site.xml
	
		<configuration>
			<property>
				<name>hadoop.tmp.dir</name>
				<value>/opt/hadoop/tmp</value>
			</property>
			<property>
				<name>fs.defaultFS</name>
				<value>hdfs://master:8020</value>
			</property>
		</configuration>
	
	3) 修改hdfs-site.xml
	
		vim /opt/hadoop-2.6.5/etc/hadoop/hdfs-site.xml
	
		<configuration>
			<property>
				<name>dfs.replication</name>
				<value>1</value>
			</property>
		</configuration>
	
	4) 配置mapred-site.xml
	
		复制模板文件并配置
	
		cp /opt/hadoop-2.6.5/etc/hadoop/mapred-site.xml.template /opt/hadoop-2.6.5/etc/hadoop/mapred-site.xml
	
		配置文件内容
	
		vim /opt/hadoop-2.6.5/etc/hadoop/mapred-site.xml
	
		<configuration>
			<property>
				<name>mapreduce.framework.name</name>
				<property>yarn</property>
			</property>
		</configuration>
	
	5) 配置yarn-site.xml
	
		vim /opt/hadoop-2.6.5/etc/hadoop/yarn-site.xml
		<configuration>
			<property>
				<name>yarn.resourcemanager.address</name>
				<value>master:8032</value>
			</property>
			<property>
				<name>yarn.resourcemanager.resource-tracker.address</name>
				<value>master:8031</value>
			</property>
			<property>
				<name>yarn.resourcemanager.scheduler.address</name>
				<value>master:8030</value>
			</property>
			<property>
				<name>yarn.nodemanager.aux-services</name>
				<value>mapreduce_shuffle</value>
			</property>
			<property>
				<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
				<value>org.apache.hadoop.mapred.ShuffleHandler</value>
			</property>
		</configuration>
	
	6) 配置从节点
	
		vim /opt/hadoop-2.6.5/etc/hadoop/slaves
		
		//能像下面这样写的前提是在hosts文件中将其IP地址配好
		
		slave1
		slave2

4. 将配置好的hadoop拷贝到从节点

		scp -r /opt/hadoop-2.6.5 root@slave1:/opt
		scp -r /opt/hadoop-2.6.5 root@slave2:/opt

5. 将环境变量拷贝到从节点

		scp -r /etc/profile root@slave1:/etc/profile
		scp -r /etc/profile root@slave2:/etc/profile

6. 格式化hdfs

		hdfs namenode -format

		注：如果不是首次进行格式化，需要删除本地hadoop的tem目录下的所有内容，再进行格式化
7. 启动hadoop

		start-dfs.sh
		start-yarn.sh

8. 查看hadoop运行情况
   1. 每个主机使用jps命令查询
   2. 浏览器访问 http://master:50070

#### 5. 测试Hadoop运行

1. 创建一个临时文件hello

		vim hello

		hello world
		hello hadoop
		hadoop

2. 将文件上传到hdfs上

		hdfs dfs -put hello /

3. 查看文件是否正确上传

		hdfs dfs -ls /

3. 对文件进行词频统计

		hadoop jar /opt/hadoop-2.6.5/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.5.jar wordcount /hello /out

	查看词频统计结果

		hdfs dfs -cat /out/part-r-00000

4. 删除本例用的hello文件和out文件夹

		hdfs dfs -rm /hello
		hdfs dfs -rm -r /out


#### 6. 关闭Hadoop


	stop-dfs.sh
	stop-yarn.sh
   
	使用jps查看进程，以上进程都已关闭。


#### 7. 关闭安全模式：
	hdfs dfsadmin -safemode leave