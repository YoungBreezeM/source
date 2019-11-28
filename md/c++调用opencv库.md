ubuntu c++ 调用 opencv 库编译 易错点 ：容易把倒引号(``)写成 " "

```
g++ test.cpp -o test `pkg-config opencv --cflags --libs opencv`
```

该方法在编译的时候动态调用opencv库

数莓派中opencv c++编译方法

CMake是一种跨平台编译工具，比make更为高级，使用起来要方便得多。CMake主要是编写CMakeLists.txt文件，然后用cmake命令将CMakeLists.txt文件转化为make所需要的makefile文件，最后用make命令编译源码生成可执行程序或共享库（so(shared object)）。

创建 CMakeLists.txt 代码如下

```
cmake_minimum_required(VERSION 2.6) #cmake 环境版本
project(test) #要编译的项目名称
find_package(OpenCV REQUIRED)#查找Opencv 库
add_executable(test  test.cpp)# 添加要编译目标文件和要输出文件
target_link_libraries(test ${OpenCV_LIBS}) #获取动态链结库

```

```
#编译
cmake . 

make
```

