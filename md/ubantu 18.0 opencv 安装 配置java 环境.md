# ubantu 18.0 opencv 安装 配置java 环境 

### 信息：

opencv2.4.1

> #提示这里不推荐使用最新版本（3.x或者4.x）因为这几个新版的不稳定，无法生成对应的.jar 包 
>
> 推荐使用2.4 （稳定版本）

java 8.0

## 1、需要安装宾并置java环境

## 2、安装环境依赖

```
sudo apt-get install cmake  
sudo apt-get install build-essential libgtk2.0-dev libavcodec-dev libavformat-dev libjpeg.dev libtiff4.dev libswscale-dev libjasper-dev
```

## 3、下载 opencv source 文件

## 4、解压文件

```
unzip opencv-3.4.3.zip
cd opencv-3.4.3
```

## 5、进入文件夹里面

```
cd ~/opencv
mkdir build
cd build
```

## 6、在build目录打开终端                                                        

```
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
make
sudo make install
```

## 7、安装opencv-java

1) 首先, 需要安装ant:

```
sudo apt-get install ant
```

2) 其次 需要运行cmake, 关键是cmake的参数, 官网给的是这样的:

```
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -DBUILD_TESTS=OFF ..
```

3) 用上面那样的参数运行cmake以后, 再:

```
make -j8
sudo make install
```

然后build 的bin 文件夹下会产生  opencv-xx.jar 这样就成功了

## 8、idea 环境开发opencv

1、导入opencv  java 的开发包

2、配置运行环境

```
-Djava.library.path=/home/yqf/文档/opencv-2.4.13.6/build/lib
```

 cmake -D CMAKE_BUILD_TYPE=RELEASE \

  -D CMAKE_INSTALL_PREFIX=/usr/local \

  -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-4.0.0/modules \

  -D ENABLE_NEON=ON \

  -D ENABLE_VFPV3=ON \

  -D BUILD_TESTS=OFF \

  -D OPENCV_ENABLE_NONFREE=ON \

  -D INSTALL_PYTHON_EXAMPLES=OFF \

  -D BUILD_EXAMPLES=OFF ..