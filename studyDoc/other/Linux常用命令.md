## 通过进程查找安装路径
	ps -ef|grep redis
	得到了进程号 xxxx
	然后 ls -l /proc/xxxx/cwd
## 查看当前所在路径
	pwd
## 和服务有关的
	service 服务名 start/stop/restart
	
## PATH 有关：
	1. 查看 PATH ： echo $PATH
	2. 修改PATH：
		1. export PATH=/usr/local/bin:$PATH   配置完后会立即生效，但只在当时的终端窗口中有效
		2. 修改 /etc/profile 中的 export PATH ， 然后运行 source /etc/profile 永久有效
## 搜索有关：
	1. whereis 文件名/命令 查看文件名或者命令所在路径
	2. which 命令 在PATH中查找命令所在路径
	3. find 目录 -name 文件名 在目录中查找文件
## 创建快捷方式：
ln命令用于将一个文件创建链接,链接分为软链接(类似于windows系统中的快捷方式)和
硬链接(相当于对源文件copy,程序或命令对该文件block的另一个访问路口)，命令默认使用硬链接。
	ln [选项] 源文件/源目录 模板路径
	选项中： -s 为创建软连接
用户相关：
	1. 在root下登录其他用户： su - 用户名