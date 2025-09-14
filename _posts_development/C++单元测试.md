---
title: C++单元测试
date: 2021-09-27 09:44:22
tags: 
categories:
   - [C/C++, 基础]
# keywords:
description:
top: #True #顶置
top_img: #（除非特定需要，可以不写）
comments: # 是否显示评论（除非设置false,可以不写）
cover: https://cdn.jsdelivr.net/gh/Dragonliu2018/FigureBed@master/img/ddddd.jpg
toc: true #是否显示toc （除非特定文章设置，可以不写）
toc_number: #是否显示toc数字 （除非特定文章设置，可以不写）
copyright: #是否显示版权 （除非特定文章设置，可以不写）
---

# 0 情景导入

在实习时，用到了C++中的单元测试，整理一下。分别在Windows和子系统（Ubuntu）中安装配置。

# 1 安装googletest

## 1.1 环境配置

1. 下载gtest：`git clone https://github.com/google/googletest.git`

2. 安装cmake：

   > CMake 是一个开源的跨平台的自动化构建系统，主要用于 C++ 的工程构建、测试以及打包等自动化操作。它能根据开发者编写的 CMakeLists.txt 规则文件，在不同的平台输出所需要的工程文件，然后开发者可以按照常见的构建方式使用生成的工程文件编译最终的程序。
   >
   > CMake 是 Cross platform Make 的缩写，虽然名字中含有“make”，但是 CMake 和 Linux 上常见的 make 系统有所区别，是更上一层的构建系统。

   * **Windows**：首先打开 CMake 官方网站，在[下载页面](https://cmake.org/download/)选择 Windows win64-x64 Installer 下载并安装
   * **Ubuntu**：`apt-get install cmake`

3. windows配置clion编译环境：[Window10上CLion极简配置教程](https://www.jianshu.com/p/1aa989808e15)

## 1.2 编译gtest库

1. 使用clion打开`googletest`代码目录，选择`gtest`项目编译生成：

   <img src="https://i.loli.net/2021/10/06/quPZfVpBdWGCriI.png" width = "800" height = "300" alt="图片名称" align=center id=29 />

2. 显示下面的信息表明编译成功（显示`Error running`，不用管他），生成的`libgtestd.a`即为gtest的库文件，项目中引用这个库文件就能使用gtest了。

   <img src="https://i.loli.net/2021/10/06/c2JyQ41X3nImkTK.png" width = "900" height = "200" alt="图片名称" align=center id=30 />

# 2 使用googletest

## 2.1 引入库

新建测试项目`cliontest`，将libgtestd.a文件拷贝到该代码根路径的lib路径下，在CMakeList.txt中加上以下内容：

```cmake
# 添加上库文件的路径，注意相对路径
link_directories(lib/)
# 添加可执行文件
add_executable(cliontest main.cpp)
# 链接gtest库文件
target_link_libraries(cliontest libgtestd.a)
```

## 2.2 引入头文件

拷贝`googletest/include`下的`gtest`目录到项目下的`include`目录下，然后在CMakeList.txt中添加上对应的调用：

```cmake
include_directories(
    include/
)
```

然后在代码中添加头文件gtest/gtest.h就可以使用了。

## 2.3 测试

修改`main.cpp`并运行：

```c++
#include "gtest/gtest.h"
 
int add(int a, int b) {
    return a + b;
}
 
TEST(add, zero) {
    EXPECT_EQ(0, add(0, 0));
}
 
TEST(add, positive_number) {
    EXPECT_EQ(3, add(1, 2));
}
 
TEST(add, negative_number) {
    EXPECT_EQ(-3, add(-1, -2));
}
 
int main() {
    ::testing::InitGoogleTest();
    return RUN_ALL_TESTS();
}
```

# 3 gtest的使用教程

参考文档：[Googletest Primer](https://github.com/google/googletest/blob/master/googletest/docs/primer.md)，google官方出品。

https://www.dyxmq.cn/program/usage-of-googletest.html

# 4 参考

* [CMake 安装指南](https://meishizaolunzi.com/cmake-an-zhuang-zhi-nan/)
* 