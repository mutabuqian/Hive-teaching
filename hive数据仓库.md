# Hive数据仓库

[Markdown Online - 专业在线 Markdown 编辑器](https://www.markdownonline.net/zh/)

## 安装jdk

```shell
cd /usr/lib
sudo mkdir jvm
sudo wget https://repo.huaweicloud.com/java/jdk/8u151-b12/jdk-8u151-linux-x64.tar.gz
sudo tar -zxvf jdk-8u151-linux-x64.tar.gz -C /usr/lib/jvm
vim ~/.bashrc
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_151
export PATH=${JAVA_HOME}/bin:$PATH
source ~/.bashrc
```

## 安装Hadoop  

```shell
 
 #解压
 sudo tar -zxf hadoop-3.3.4.tar.gz -C /usr/local
 cd /usr/local/
 #重命名
 sudo mv hadoop-3.3.4/ hadoop
 #修改拥有者
 sudo chown -R zly hadoop/
 
 #配置环境变量
 vim ~/.bashrc
 export HADOOP_HOME=/usr/local/hadoop 
 export PATH=${JAVA_HOME}/bin:$PATH:$HADOOP_HOME/bin
 
 #测试
 hadoop version
 
 #hadoop 要用到java，配置Hadoop-env
 vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh
 
 export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_151
 export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop

```

## 部署Hadoop伪分布式集群

```shell

vim core-site.xml
```

```xml

<property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/hadoop/tmp</value>
    </property>


```

```shell
vim hdfs-site.xml
```

```xml
 <property>
        <name>dfs.namenode.name.dir</name>
        <value>/usr/local/hadoop/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/usr/local/hadoop/dfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>

```

```shell
vim mapred-site.xml	
```

```xml
<property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property>
```

```shell
vim yarn-site.xml
```

```xml
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>Master</value>
</property>
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
        <name>yarn.application.classpath</name>
    <value>
    /usr/local/hadoop/etc/hadoop:/usr/local/hadoop/share/hadoop/common/lib/*:/usr/local/hadoop/share/hadoop/common/*:/usr/local/hadoop/share/hadoop/hdfs:/usr/local/hadoop/share/hadoop/hdfs/lib/*:/usr/local/hadoop/share/hadoop/hdfs/*:/usr/local/hadoop/share/hadoop/mapreduce/*:/usr/local/hadoop/share/hadoop/yarn:/usr/local/hadoop/share/hadoop/yarn/lib/*:/usr/local/hadoop/share/hadoop/yarn/*
        
    </value>
</property>
```

```shell
hdfs namenode -format
start-dfs.sh
start-yarn.sh
jps
stop-dfs.sh
stop-yarn.sh
```

## 安装Hive

```shell
sudo tar -zxf apache-hive-4.0.1-bin.tar.gz  -C /usr/local/
sudo mv apache-hive-4.0.1-bin/ hive
sudo chown -R zly hive/
sudo vim ~/.bashrc
```

```bash
export HIVE_HOME=/usr/local/hive
export PATH=${JAVA_HOME}/bin:$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin
```

```shell
source ~/.bashrc
cd hive/
cp conf/hive-env.sh.template conf/hive-env.sh
vim conf/hive-env.sh
```

hive要用到Hadoop

```bash
export HADOOP_HOME=/usr/local/hadoop
export HIVE_CONF_DIR=/usr/local/hive/conf
```

启动derby

```shell
schematool -initSchema -dbType derby
```

## 安装MySQL

```shell
#acquire MySQL connector
sudo rpm2cpio mysql-connector-j-8.0.33-1.el9.noarch.rpm | cpio -idmv
#install mysql server
sudo yum install mysql-server
#start and autostart mysql
sudo systemctl start mysqld
sudo systemctl enable mysqld
systemctl daemon-reload
#mysql status
systemctl status mysqld
#look up username and password
sudo cat var/log/mysql/mysqld.log
#connect mysql
mysql -h localhost -u root
#update the password
alter user 'root'@'localhost' identified by '123456';
```





## 配置MySQL保存Hive元数据

```shell

cp mysql-connector-j.jar /usr/local/hive/lib/
```

配置hive-site.xml

```shell
vim hive-site.xml
```

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<!--连接数据的用户名-->
  <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>root</value>
  </property>
<!--连接数据的密码-->
  <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>172741</value>
  </property>
<!--mysql数据库的访问路径，没有路径则自动创建-->
  <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
  </property>
<!--连接数据库的驱动-->
  <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.cj.jdbc.Driver</value>
</property>
<!--元数据是否校验-->
  <property>
      <name>hive.metastore.schema.verification</name>
      <value>false</value>
  </property>
</configuration>

```



```shell
#元数据存储在mysql
schematool -initSchema -dbType mysql
#测试hive连接mysql
mysql -u root -p
show databases;

```

## 网络配置

分别更改三台虚拟机的主机名为**Master，Worker1，Worker2**

```shell
#update hostname 
sudo vim /etc/hostname
#虚拟机Master
Master
#虚拟机Worker1
Worker1
#虚拟机Worker2
Worker2
```

hosts 文件配置，将IP和主机名映射，都要在三台虚拟机配置

```shell
sudo vim /etc/hosts

192.168.1.11 Master
192.168.1.12 Worker1
192.168.1.13 Worker2

```



## 免密登录

三台虚拟机可以ping通，如何互相操作？用ssh连接控制，ssh每次需要密码，设置**免密登录**无需密码

use key pair, public key and private key, the place to store the key

```shell
#generate key
ssh-keygen -t rsa
#authorize the public key
cat id_rsa.pub  >> authorized_keys
#distribute public key
scp id_rsa.pub zly@Worker1:/home/zly
scp id_rsa.pub zly@Worker2:/home/zly
#append the pk to the authorized file
cat id_rsa.pub >> ./.ssh/authorized_keys
#test
ssh Worker1
#exit
exit
```

## 部署Hadoop完全分布式集群

之前伪分布就用到一台虚拟机，真正的Hadoop应该在多台虚拟机部署，此处在三台虚拟机部署，他们的主机名分别为Master，Worker1，Worker2

三台虚拟机都要安装jdk和Hadoop

在**Master虚拟机**配置workers文件：在为Master，Worker1，Worker部署，但是它们自己不知道，得告诉它们，怎样告诉？修改配置文件。文件在哪？

```shell
#文件在哪
cd /usr/local/hadoop/etc/hadoop/
vim workers
#清空文件内容后添加下面信息
Master
Worker1
Worker2

```

在**Master虚拟机上**分别配置core-site.xml，hdfs-site.xml，mapred-site.xml，yarn-site.xml，这些文件在哪？

```shell
#文件在哪？
cd /usr/local/hadoop/etc/hadoop/
```

core-site.xml

```xml
<property>
        <name>fs.defaultFS</name>
        <value>hdfs://Master</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/hadoop/tmp</value>
    </property>
   <property>
     <name>hadoop.proxyuser.zly.hosts</name>
     <value>*</value>
   </property>
   <property>
     <name>hadoop.proxyuser.zly.groups</name>
     <value>*</value>
   </property>
```

hdfs-site.xml

```xml
<property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/dfs/name</value>
 </property>
 <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/dfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
<name>dfs.nameservices</name>
<value>Master</value>
</property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>Master:50090</value>
    </property>
```

mapred-site.xml



```xml
<property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property>

```

yarn-site.xml

```xml
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>Master</value>
</property>
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
        <name>yarn.application.classpath</name>
    <value>
    /usr/local/hadoop/etc/hadoop:/usr/local/hadoop/share/hadoop/common/lib/*:/usr/local/hadoop/share/hadoop/common/*:/usr/local/hadoop/share/hadoop/hdfs:/usr/local/hadoop/share/hadoop/hdfs/lib/*:/usr/local/hadoop/share/hadoop/hdfs/*:/usr/local/hadoop/share/hadoop/mapreduce/*:/usr/local/hadoop/share/hadoop/yarn:/usr/local/hadoop/share/hadoop/yarn/lib/*:/usr/local/hadoop/share/hadoop/yarn/*
        
    </value>
</property>
```

在**Master虚拟机**将/**etc/hadoop/**下的文件复制到**Worker1**和**Worker2**中

```shell
scp -r /usr/local/hadoop/etc/hadoop/* Worker1:/usr/local/hadoop/etc/hadoop/
scp -r /usr/local/hadoop/etc/hadoop/* Worker2:/usr/local/hadoop/etc/hadoop/
```

在**Master虚拟机**格式化**namenode**

```shell
hdfs namenode -format
```

在**Master虚拟机**启动**hdfs**和**Yarn**

```shell
start-dfs.sh
start-yarn.sh
```

查看进程

```shell
#master
jps

```

![image-20250916142039484](C:\Users\17274\Documents\zly\亚视演艺\25-26 第1学期\hive数据仓库\选用\assets\image-20250916142039484.png)

```shell
#Worker1,Worker2
jps
```



## 部署HiveServer2和Beeline

master、Worker1、Worker2虚拟机上的**core-site.xml**添加如下内容

```xml
<property>
     <name>hadoop.proxyuser.zly.hosts</name>
     <value>*</value>
   </property>
   <property>
     <name>hadoop.proxyuser.zly.groups</name>
     <value>*</value>
   </property>
```

重启集群

```
stop-all.sh
start-dfs.sh
start-yarn.sh
```

在Worker1用本地模式部署Hive

Worker2中安装Hive

修改Worker1上的MySQL权限，使得任何主机可以访问Worker1上的Mysql

```shell
#login in mysql
mysql -u root -p
#enter password
```

修改权限

```sql
use mysql;
update user set host="%" where user="root";
flush privileges;
```

启动HiveServer2服务

```shell
hiveserver2
```

是否正常启动

```shell
jps
```

![image-20250916141944729](C:\Users\17274\Documents\zly\亚视演艺\25-26 第1学期\hive数据仓库\选用\assets\image-20250916141944729.png)

**Worker2**元数据存在**服务**端**Worker1**中：关闭本地meta store服务，连接Worker1

```shell
cd /usr/local/hive/conf/
#打开该文件，若不为空则清空，然后添加以下信息
vim hive-site.xml
```

```xml
 1 <?xml version="1.0" encoding="UTF-8" standalone="no"?>
  2 <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  3 <configuration>
  4   <property>
  5       <name>hive.metastore.local</name>
  6       <value>false</value>
  7   </property>
  8   <property>
  9       <name>hive.metastore.uris</name>
 10       <value>thrift://Worker1:9083</value>
 11   </property>
 12 </configuration>

```



Worker1启动Beeline连接Worker2的HiveServer2服务

```shell
beeline --hiveconf hive.server2.logging.operation.level=NONE -u jdbc:hive2://Worker1:10000 -n zly -p
```

