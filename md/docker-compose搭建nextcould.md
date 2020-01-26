## Nextcloud docker 部署

1、更新软件

```
yum update
```

2、安装必要工具

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

3、docker安装软件源

```shell
yum-config-manager --add-repo=https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

4、更新yum缓存，安装docker

```shell
yum makecache
yum install docker-ce
```

5、设置开机自动启动、启动docker

```
systemctl enable docker
systemctl start docker
```

5.1、配置镜像加速

```
vim /etc/docker/daemon.json
```

添加以下内容

```json
{  
"debug": true, 
"experimental": false,  
"registry-mirrors": [   
      "https://o5ot2rw5.mirror.aliyuncs.com", 
       "https://registry.docker-cn.com"
       ]
}
```

> 插一句
> 阿里云的镜像加速地址，需要先注册阿里云之后才能看到，注册之后访问https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors就可以看到了,要对其中地址进行更新

5.2、重启docker

```
systemctl restart docker
```

6、安装docker-compose 搭建镜像网络

```shell
yum install docker-compose
```

7、新建一个docker-compose.yml，只配置web系统内容如下

```yml
version: '2'
services:
  app:  
    image: nextcloud
    restart: always
   ports:
     - 80:80
   volumes:
     - /home/data/nextcloud/www:/var/www/html
```

配置web系统加mysql

```yml
version: '2'
services:
  db:
    image: mariadb
    restart: always
    volumes:
      - /home/data/nextcloud/db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_PASSWORD=nextcloud
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
  app:  
    image: nextcloud
    restart: always
    ports:
      - 80:80
    links:
      - db
    volumes:
      - /home/data/nextcloud/www:/var/www/html
```

8、运行网络容器、会自动下载配置文件中的镜像

```
docker-compose up -d
```

9、可能用到的命令

停止所有容器

```shell
docker stop $(docker ps -a -q)
```

删除所有容器

```shell
docker rm $(docker ps -a -q)
```

删除所有镜像

```shell
docker rmi $(docker images -q)
```

<u>注意：</u>如果你之前已经创建网络，需要把只之前生成的文件删除，才能重新配置，或者配置到不同的目录下

创建容器网络

```sh
docker-compose up -d
```

## Nextcloud 源码本地直接部署

