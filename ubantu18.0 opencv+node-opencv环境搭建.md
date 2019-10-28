nodejs 人脸识别 opencv 环境搭建 

# ubantu18.0 opencv3.4 安装



## 1.1 下载Opencv 3.4.3

去官网下载opencv，在本教程中选用的时opencv3.4.3，其他版本的配置方法异曲同工。
下载链接 http://opencv.org/releases.html，选择sources版本。



## 1.2 解压zip包

```css
unzip opencv-3.4.3.zip
cd opencv-3.4.3
```



## 1.3 安装依赖库和cmake

```css
sudo apt-get install cmake  
sudo apt-get install build-essential libgtk2.0-dev libavcodec-dev libavformat-dev libjpeg.dev libtiff4.dev libswscale-dev libjasper-dev
```

> #注意这边libjasper-dev 可能会安装失败导致cmake 的时候java模块编译失败
>
> ```
> sudo add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
> sudo apt update
> sudo apt install libjasper1 libjasper-dev
> ```
>
> 一定要注意这个依赖是否安装成功否着cmake 编译一定要注意
>
> ## 1.4 执行cmake
>

```cpp
/* 新建编译文件夹*/
mkdir build
cd build
/* 执行cmake */
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..  
```



## 1.5 执行make命令

```go
sudo make  // 时间比较漫长
```



## 1.6 执行install命令

```go
sudo make install
```

这一步执行完毕之后，Opencv的编译过程就结束了，接下来的工作就是配置一些Opencv的编译环境。



## 2.1 将OpenCV的库添加到路径

用gedit打开/etc/ld.so.conf
在文件中加上一行 /usr/loacal/lib
其中/user/loacal是opencv安装路径也就是makefile中指定的安装路

```
sudo gedit /etc/ld.so.conf
```

在最下面加一个

usr/local/lib

运行

```
sudo ldconfig
```

修改bash.bashrc文件

```
sudo gedit /etc/bash.bashrc 
```

在文件末尾加入：
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig
export PKG_CONFIG_PATH



然后在命令行中输入

```
source /etc/bash.bashrc
```

## 2.2检验

在命令行中输入如下命令：

```
pkg-config opencv --modversion
```

会提示版本信息 则安装成功

### 以上属于环境搭建部分



# node-opencv 安装

node-opencv 具体操作去npm包管理社区查看

1、先去github上下载node-opencv 开发工具包  https://github.com/peterbraden/node-opencv

2、npm install #安装packjson.json 里的依赖

3、安装  npm install opencv



然后就可以使用opencv 了