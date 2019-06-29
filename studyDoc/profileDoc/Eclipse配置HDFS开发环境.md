# Eclipse配置HDFS开发环境
**为了在Eclipse上远程调试MapReduce，需要提前在Eclipse上配置HDFS：**

> 补充说明：在Windows端才需要添加此`winutils.exe`，Mac端不需进行2、3、4步

1. 在XShell上下载集群上的Hadoop的配置文件：```sz /opt/hadoop-2.6.5/etc/hadoop/etc```，`sz filename`是在Xshell上下载远程终端文件的命令
2. 在本地解压一份 Hadoop 的文件，将etc文件放入` /shixun `文件夹下
3. 将 `hadoop-common-2.2.0-bin-master` 解压到 `/shixun` 下
4. 在 `hadoop-common-2.2.0-bin-master/bin` 中找到 `winutils.exe` 复制到 `/shixun/hadoop-2.6.5/bin` 中
5. 
