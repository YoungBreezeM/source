# 一、安装centos系统

1、选择安装空间（自动挂载）

2、开启网络连接（在虚拟机安装的时候选者桥连接）

3、配置root密码和用户、密码

项目环境配置

1、更新软件包

```
yum update
```

```
yum install wget
```

# 二、项目运行环境部署

### 1、安装java jdk环境

```
yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel
```

  

### 2、配置环境变量

```
vi  /etc/profile  #修改环境变量
#环境配置
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b10-1.el7_7.x86_64
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile #让环境立即生效
```

运行   `java -version`  出现以下信息java环境配置成功

#openjdk version "1.8.0_222"
#OpenJDK Runtime Environment (build 1.8.0_222-b10)
#OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)

#### #<u>***注意这里路径配置错误会导致tomcat启动不成功***</u>

### 3、tomcat 下载

下载tomcat7安装包 

```
wget  http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-7/v7.0.96/bin/apache-tomcat-7.0.96.tar.gz
```

apache-tomcat-7.0.91tomcat 下载地址https://tomcat.apache.org/download-70.cgi

解压安装包

```
tar -zxvf apache-tomcat-7.0.96.tar.gz
```

可以新建多个tomcat空文件夹，可以同时运行多个tomcat

**<u>注意</u>** 在centos7.3中启用多个tomcat 只要保证指定端口不重复就可以使用

第一个tomcat 可以使用默认端口配置

这里我们直接修改第二个

进入到tomcat/conf目录

```
cd tomcat/conf
```

修改server.xml

```
  vi server.xml
```



> <Server port="8005" shutdown="SHUTDOWN"> #8005修改成8806
>
> <Connector port="8080" protocol="HTTP/1.1" #8080修改成8081
> connectionTimeout="20000"
> redirectPort="8443"  URIEncoding="UTF-8" />
> #添加URIEncoding="UTF-8" 防止乱码
>
> <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>    #8009修改成8010

#系统库tomcat 服务命令#

```
#启动服务 
systemctl start tomcat.service   
#关闭服务   
systemctl stop tomcat.service   
#开机启动   
systemctl enable tomcat.service 
```

tomcat 的安装目录默认在 `/usr/share/tomcat下`

### 4、防火墙端口配置(开机自动启动)

1、查看firewall服务状态

```
systemctl status firewalld 
```

2、查看firewall的状态

```
firewall-cmd --state
```

3、开启、重启、关闭、firewalld.service服务

```
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop
```

4、查看防火墙规则

```
firewall-cmd --list-all 
```

5、查询、开放、关闭端口

```
# 查询端口是否开放
firewall-cmd --query-port=8080/tcp
# 开放80端口
firewall-cmd --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp
#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload
# 参数解释
1、firwall-cmd：是Linux提供的操作firewall的一个工具；
2、--permanent：表示设置为持久；
3、--add-port：标识添加的端口；
```

6、防火墙自动开机启动

```
systemctl status firewalld.service
```



### 5、开启每个tomcat服务

进入到tomcat\bin 启动tomcat 服务

```
./startup.sh
```



### 6、配置mysql

1、安装 mysql 源

```csharp
# 下载
shell> wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
# 安装 mysql 源
shell> yum localinstall mysql57-community-release-el7-11.noarch.rpm
```

2、安装mysql

```undefined
yum install -y mysql-community-server
```

3、启动mysql

```
systemctl start mysqld
```

4、设置开机启动

```bash
shell> systemctl enable mysqld
# 重载所有修改过的配置文件
shell> systemctl daemon-reload
```

5、生成本地密码

```bash
 grep 'temporary password' /var/log/mysqld.log
```

在localhost:" password",会跟一串密码

6、

```bash
shell> mysql -uroot -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!'; 
```

7、配置mysql 参数，原来mysql配置不允许密码配置过于简单，所以需要以下配置。

> mysql> set global validate_password_policy=0;
> Query OK, 0 rows affected (0.05 sec)
>
> mysql> 
> mysql> 
> mysql> set global validate_password_mixed_case_count=0;
> Query OK, 0 rows affected (0.00 sec)
>
> mysql> set global validate_password_number_count=3;
> Query OK, 0 rows affected (0.00 sec)
>
> mysql> set global validate_password_special_char_count=0;
> Query OK, 0 rows affected (0.00 sec)
>
> mysql> set global validate_password_length=3;
>

8、修改完之后继续修该密码

```bash
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!'; 
flush privileges;
```

9、配置远程连接

```bash
GRANT ALL PRIVILEGES ON *.* TO 'zhangsan'@'%' IDENTIFIED BY 'Zhangsan2018!' WITH GRANT OPTION;
```

10、设置默认编码

mysql 安装后默认不支持中文，需要修改编码。
修改 `/etc/my.cnf` 配置文件，在相关节点（没有则自行添加）下添加编码配置，如下：

```csharp
[mysqld]
character-set-server=utf8
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
```

重启mysql服务，查询编码。可以看到已经改过来了

```dart
shell> systemctl restart mysqld
shell> mysql -uroot -p
mysql> show variables like 'character%';
```

9、修改mysql wait_timeout 时间

```mysql
vim /etc/my.cnf 
#在mysqld 增加一行
wait_timeout=31536000
```



## 7、nginx 配置

1、添加nginx 源

```csharp
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

2、安装nginx

```csharp
yum install nginx
```

3、启动nginx

```csharp
service nginx start
```

4、重启nginx

```
nginx -s reload 
```

5、nginx 安装目录/etc/nginx

6、关闭selinux,如果不关闭centos selinux 会造成nginx 无法连接

```
setsebool -P httpd_can_network_connect 1
```

# 三、项目部署

1、先通过navicat 远程连接到数据库，新建电业数据库

运行导入文件dianye.sql

2、修改项目上对应的数据库配置，把项目的 .war 文件拖到服务器apache webapps 下面，访问ip 测试项目接口是否能用

3、修改nginx配置，配置好转接路由。把项目生成的最终文件放在html目录下





# 四、项目性能调优

### 1、tomcat8.5 配置连接数，线程数，缓存，修改server.xml

打开被注释的默认连接池配置

 默认配置：

```
<!--  
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"  
maxThreads="150" minSpareThreads="4"/>  
-->  
```

修改：

```
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"  
         maxThreads="150" 
         minSpareThreads="100"   
         prestartminSpareThreads="true" 
         maxQueueSize="100"/>   
```

参数讲解：
name: 线程名称
namePrefix: 线程前缀
maxThreads : 最大并发连接数，不配置时默认200，一般建议设置500~ 800 ，要根据自己的硬件设施条件和实际业务需求而定。
minSpareThreads：Tomcat启动初始化的线程数，默认值25   
prestartminSpareThreads：在tomcat初始化的时候就初始化minSpareThreads的值， 不设置trueminSpareThreads  的值就没啥效果了 。
maxQueueSize: 最大的等待队列数，超过则拒绝请求

修改连接配置

默认配置：

```
<Connector port="8080" protocol="HTTP/1.1" 
       connectionTimeout="20000" 
      redirectPort="8443" /> 
```

修改为

```
connectionTimeout="20000"  
        redirectPort="8443"   
        executor="tomcatThreadPool"  
        enableLookups="false"   
        maxIdleTime="60000"
        acceptCount="100"   
        maxPostSize="10485760" 
        acceptorThreadCount="2"    
        disableUploadTimeout="true"   
        URIEncoding="utf-8"
        keepAliveTimeout ="6000"  
        maxKeppAliveRequests="500"  
```

参数讲解：
port：连接端口。  
protocol：连接器使用的传输方式。  
executor： 连接器使用的线程池名称
enableLookups：禁用DNS  查询
maxIdleTime：线程空闲时间，超过该时间后，空闲线程会被销毁，默认值为6000（1分钟），单位毫秒。
acceptCount：指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理，默认设置 100 。
maxPostSize：限制 以FORM URL 参数方式的POST请求的内容大小，单位字节，默认是 2097152(2兆)，10485760 为 10M。如果要禁用限制，则可以设置为 -1。
acceptorThreadCount： 用于接收连接的线程的数量，默认值是1。一般这个指需要改动的时候是因为该服务器是一个多核CPU，如果是多核 CPU 一般配置为 2。
disableUploadTimeOut：允许Servlet容器，正在执行使用一个较长的连接超时值，以使Servlet有较长的时间来完成它的执行，默认值为false
keepAliveTimeout - 表示在下次请求过来之前，tomcat保持该连接多久。这就是说假如客户端不断有请求过来，且未超过过期时间，则该连接将一直保持。
maxKeepAliveRequests -表示该连接最大支持的请求数。超过该请求数的连接也将被关闭（此时就会5、mysql 调优返回一个Connection: close头给客户端）。 (maxKeepAliveRequests="1"代表禁用长连接)（1表示禁用，-1表示不限制个数，默认100个。一般设置在100~200之间）



### 2、JVM 调优



### 3、nginx 调优



### 4、java web 进程守护



### 5、mysql 调优

##### MySQL非缓存参数变量介绍及修改

```
/etc/my.cnf.d/server.cnf
```

### **读缓存，线程缓存，排序缓存**

sort_buffer_size = 2M
connection级参数。太大将导致在连接数增高时，内存不足。



网络传输中一次消息传输量的最大值。系统默认值 为1MB，最大值是1GB，必须设置1024的倍数。

join_buffer_size = 2M
和sort_buffer_size一样，该参数对应的分配内存也是每个连接独享

tmp_table_size = 256M
默认大小是 32M。GROUP BY 多不多的问题

max_heap_table_size = 256M
key_buffer_size = 2048M
索引的缓冲区大小，对于内存在4GB左右的服务器来说，该参数可设置为256MB或384MB。

read_buffer_size = 1M
read_rnd_buffer_size = 16M
进行排序查询时，MySql会首先扫描一遍该缓冲，以避免磁盘搜索

bulk_insert_buffer_size = 64M
批量插入数据缓存大小，可以有效提高插入效率，默认为8M

### **Innodb缓存**

innodb_buffer_pool_size = 2048M
只需要用Innodb的话则可以设置它高达 70-80% 的可用内存。一些应用于 key_buffer 的规则有 ——如果你的数据量不大，并且不会暴增，那么无需把innodb_buffer_pool_size 设置的太大了。

innodb_additional_mem_pool_size = 16M
网络传输中一次消息传输量的最大值。系统默认值为1MB，最大值是1GB，必须设置1024的倍数。

innodb_log_files_in_group = 3
循环方式将日志文件写到多个文件。推荐设置为3

innodb_lock_wait_timeout = 120
InnoDB 有其内置的死锁检测机制，能导致未完成的事务回滚。innodb_file_per_table = 0 独享表空间，关闭

##  连接数

open_files_limit = 10240
允许打开的文件数

back_log = 600
短时间内的多少个请求可以被存在堆栈中

max_connections = 3000
MySQL默认的最大连接数为100，MySQL服务器允许的最大连接数16384

max_connect_errors = 6000
设置每个主机的连接请求异常中断的最大次数，当超过该次数，MYSQL服务器将禁止host的连接请求

thread_cache_size = 300
重新利用保存在缓存中线程的数量

thread_concurrency = 8
thread_concurrency应设为总CPU核数的2倍

thread_stack = 192K
每个线程的堆栈大小，默认值足够大，可满足普通操作。可设置范围为128K至4GB，默认为192KB。

