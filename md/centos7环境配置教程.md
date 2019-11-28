## 一、java\tomcat7多傻瓜式服务配置\

### 1、安装java jdk环境

```
yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel
```

  

### 2、配置环境变量

```
vi  /etc/profile  #修改环境变量
#环境配置
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-1.el7_7.x86_64
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile #让环境立即生效
```

运行   `java -version`  出现以下信息java环境配置成功

#openjdk version "1.8.0_222"
#OpenJDK Runtime Environment (build 1.8.0_222-b10)
#OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)

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
>  #添加URIEncoding="UTF-8" 防止乱码
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



### 5、开启每个服务

进入到tomcat\bin 启动tomcat 服务

```
./startup.sh
```



### 6、多个tomcat 响应慢的问题

原因：

  Tocmat的Session ID是通过SHA1算法计算得到的，计算Session ID的时候必须有一个密钥。为了提高安全性Tomcat在启动的时候回通过随机生成一个密钥。由于没有足够的熵来产生随机数，导致速度特别慢

解决：

```
yum install rng-tools *# 安装rngd服务（熵服务，增大熵池）*
systemctl start rngd  *# 启动服务*
```

## 1.1 采用docker 部属多个独立微服务

- ## 前提条件

  目前，CentOS 仅发行版本中的内核支持 Docker。

  Docker 运行在 CentOS 7 上，要求系统为64位、系统内核版本为 3.10 以上。

  Docker 运行在 CentOS-6.5 或更高的版本的 CentOS 上，要求系统为64位、系统内核版本为 2.6.32-431 或者更高版本。

  > 参照于https://www.runoob.com/docker/centos-docker-install.html

安装一些必要的系统工具：

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

添加软件源信息：

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

更新 yum 缓存：

```
sudo yum makecache fast
```

安装 Docker-ce：

```
sudo yum -y install docker-ce
```

启动 Docker 后台服务

```
sudo systemctl start docker
```

### Docker 安装 Tomcat（文件挂在）

```
docker pull tomcat
```

查看镜像是否安装

```
docker images
```

\#查看进入容器的路径

```
docker exec -it 10e723ab7004 /bin/bash
```

在宿主主机里新建一个文件夹用来存放要挂在到容器tomcat下的webpapp

把宿主 /usr/local/src/webapps/yqf.war 的文件放到容器/usr/local/tomcat/webapps/yqf.war 下 yqf.war 会自动解压成yqf 的一个文件夹

```
docker run -d -v /usr/local/src/webapps/yqf.war:/usr/local/tomcat/webapps/yqf.war -p 8080:8080 docker.io/tomcat
```

启动多个tomcat 只要端口不重复就可使用

```
# 注意每次启动容器 docker.io 也就是这容器实例名字不应该重复 -p 宿主机端口号（对外开放的端口号）：容器端口的默认号（在制作容器镜像可知自己设置）
docker run -d -v /usr/local/src/webapps/yqf.war:/usr/local/tomcat/webapps/yqf.war -p 8080:8080 docker.io/tomcat

```

### Docker 安装 Tomcat（centos7-jdk8-tomcat镜像制作）

新建一个文件夹tomcat用来存放tomcat 和 jdk 用于镜像制作

包文件可以通过远程连接工具上传到该目录下

```
mkdir /usr/local/src/tomcat
```

```
# 注意 创建以下 新建文件必须以 Dockerfile  命名 如果名字不对制作镜像将找不到相关配置
vi Dockerfile 
```

```
#把下面内容赋值到 Dockerfile  里面
FROM centos:centos7.6.1810
MAINTAINER qingfeng 940695836@qq.com
ADD jdk-8u231-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-7.0.96.tar.gz /usr/local/
ENV JAVA_HOME /usr/local/jdk1.8.0_231
ENV CATALINA_HOME /usr/local/apache-tomcat-7.0.96
ENV PATH $PATH:$JAVA_HOME/bin:$CATCHA_HOME/bin
EXPOSE 8080
CMD ["/usr/local/apache-tomcat-7.0.96/bin/catalina.sh","run"]
```

- 参数说明

- FROM: 指定基础镜像，并且必须是第一条指令
  MAINTAINER： 指定作者
  RUN: 运行指定的命令
  ADD: 复制命令，把文件复制到镜像中，压缩文件会自动解压。
  ENV: 设置环境变量
  EXPOSE: 功能为暴漏容器运行时的监听端口给外部
  CMD: 指定容器启动时运行的命令

```
#在有Dockerfile 文件里面 运行该命令
docker build -t yqf/tomcatyqf .
```

`-t` 设置`tag`名称, 命名规则`registry/image:tag`（若不添加版本号,默认`latest`）
`.` 指定镜像构建上下文

查看镜像是否制作成功

```
docker images 
```

运行镜像

```
docker run -d -p 8081:8080 yqf/tomcatyqf
```

> `-d` 后台运行
> `-p` 端口映射 宿主机port : 容器port

## 二、团队资料共享服务器confluence配置（内存大小必须大于4g,内存过小会造成安装失败）

> 参照与https://blog.csdn.net/qq_34889607/article/details/81118106

### 1、安装mysql 数据库

```
sudo yum install -y mariadb mariadb-server

sudo systemctl start mariadb

 sudo systemctl enable mariadb
```

### 2、 修改配置文件

```
sudo vi /etc/my.cnf                      # 在[mysqld]下面添docker run -d -p 8083:8080 www19930327/centos7-jdk8-tomcat加如下

init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
transaction-isolation=READ-COMMITTED
```

```
sudo vi /etc/my.cnf.d/mysql-clients.cnf ---> # 在[mysql]下面添加如下
  
default-character-set=utf8
```

```
sudo vi /etc/my.cnf.d/client.cnf ---> # 在[client]下面添加如下
default-character-set=utf8

```

### 3、给mariadb设置密码，并简单设置

```
$ sudo mysql_secure_installation

​```
# 这里就自己根据提示一步一步来就可以了
​```
$ sudo mysql -u root -p

​```
Enter password:  ---> 输入密码
​```

MariaDB [(none)]> show variables like '%character%';

MariaDB [(none)]> show variables like '%collation%';
MariaDB [(none)]> create database confluence default character set utf8 collate utf8_bin;

MariaDB [(none)]> grant all on confluence.* to 'admin'@'%' identified by 'admin';

MariaDB[(none)]> grant all on confluence.* to 'admin'@'localhost' identified by'admin';

MariaDB [(none)]> flush privileges;

MariaDB [(none)]> select user,host,password from mysql.user;

```

```
重启数据库
sudo systemctl restart mariadb
```



## **4、安装confluence**

1、下载confluence 

```
wget https://product-downloads.atlassian.com/software/confluence/downloads/atlassian-confluence-6.10.0-x64.bin
```

2、

```
chmod +x atlassian-confluence-6.10.0-x64.bin #授予执行权限
```

3、

```
./atlassian-confluence-6.10.0-x64.bin #安装
```



    o
    
    …………
    
    Express Install (uses default settings) [1],
    
    Custom Install (recommended for advanced users) [2, Enter],
    
    Upgrade an existingConfluence installation [3] ---> 输入1
    
    1
    
    …………
    
    Install [i, Enter], Exit [e] ---> 输入i
    
    i
    
    …………
    
    Yes [y, Enter], No[n] ---> 输入y
    
    y
    
    …………
可以访问 服务器地址：8090 可以完成项目初始化

### **5、破解confluence**

##### /home/hy/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar”这个文件拷贝到windows上面，改名为“atlassian-extras-2.4.jar

然后在windows下解压“confluence5.6.6-crack.zip”这个破解包，里面有个“confluence_keygen.jar 打开

Name和Email随意填，Organization默认的就好，Server ID就是刚才的服务器ID。

点击“.patch!”选择刚才拷贝到windows的文件“atlassian-extras-2.4.jar”，然后点击“.gen!

控制破解完成之后会生成一个新的“atlassian-extras-2.4.jar”

把新生成的“atlassian-extras-2.4.jar”文件拷贝到“/home/hy/atlassian/confluence/confluence/WEB-INF/lib/

并改名为“atlassian-extras-decoder-v2-3.4.1.jar

把mysql-connect-java驱动一起放进去

然后重新启动confluence

$ /home/hy/atlassian/confluence/bin/stop-confluence.sh

$ /home/hy/atlassian/confluence/bin/start-confluence.sh

## 三、版本控制服务

使用github 控制

