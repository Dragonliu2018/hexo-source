---
title: Clion项目配置教程
tags:
categories:
  - 环境与工具
  - [C/C++, 基础]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-17 00:31:32
---

# 1 下载clion，申请license

1. 在clion官网下载安装包（在华为实习时是在IT工具库下载的）；
2. 申请license，学生邮箱免费使用（公司里面需要审批申请license）；

# 2 第一次打开clion前修改缓存位置

1. 打开下载目录`bin`文件夹下`idea.properties`文件；

2. 放开如下注释，并修改`idea.config.path`和`idea.system.path`的取值（事先创建这两个目录）：

   ```
   idea.config.path=目录1
   idea.system.path=目录2
   idea.plugins.path=${idea.config.path}/plugins
   idea.log.path=${idea.system.path}/log
   ```

这一步是为了避免C盘存放JetBrains的工程缓存内容而被过度占用。

如果clion已经打开了，也可以修改此配置，只不过可能要重新配置一下clion的初始化步骤。

# 3 下载01_RB代码，用clion打开

1. 打开部门主仓库；
2. Fork自己的仓库；
3. 安装git并配置，然后将自己的仓库拉到本地。

# 4 手动代理设置

1. 点击`文件(File)`，选择`设置(setting)`，选择`外观和行为——系统设置——HTTP代理`
2. 进行配置：
   * 主机名：`proxy.huawei.com`
   * 端口：`8080`
   * 不为以下项使用代理：`127.0.0.1, 10.*.100.*, *.huawei.com`

<img src="https://s2.loli.net/2022/04/17/LdPivWetjscnFD9.png" width = "800" height = "400" alt="图片名称" align=center id=1 />

# 5 安装插件

1. 打开clion，点击`文件(File)`，选择`设置(settings)`，选择`插件(plugins)`；
2. 安装以下插件：Chinese，（JetBrains Foundation，Huawei Jetbrains Marketplace，codecheck未找到）
3. 重启IDE生效

# 6 配置远程工作模式

1. **配置工具链**：点击`文件(File)`，选择`设置(setting)`，选择`构建、执行、部署——工具链`，添加远程主机，配置远程主机信息。凭据需要点击后方齿轮进行配置。Make，C编译器，C++编译器会自动检测，不需要配置。

   <img src="https://s2.loli.net/2022/04/17/1u7gXWixpfLIYHT.png" width = "1000" height = "500" alt="图片名称" align=center id=2 />

2. **配置CMake**：

   * CMake 选项：`-DCMAKE_TOOLCHAIN_FILE=../cmake/scripts/gcc_ut_toolchain.cmake ..`
   * 构建目录：`cmake-build-debug`（需要与自己项目中的目录对应）
   * 构建选项：`-- -j 32`（多线程）
   * 环境：`CHG_HOME=xxx;LD_LIBRARY_PATH=xxx`（需要和自己项目中的目录对应）

3. **配置部署**：点击`文件`，选择`设置`，选择`部署`，点击`映射`，配置部署路径：`/usr1/dragon/01_RB`（改成自己的，远程主机上的项目根目录）

   <img src="https://s2.loli.net/2022/04/17/2h1tioH4Q5snyrp.png" width = "1000" height = "300" alt="图片名称" align=center id=3 />

4. **Debug配置**：登陆到远程主机，将环境变量添加到`~/.bashrc`

   ```sh
   export CHG_HOME=xxx
   export LD_LIBRARY_PATH=xxx
   export TZ=BEST-8
   export BILLING_HOME=xxx
   ```

# 7 clang-format的使用

1. 将`.clang-fromat`文件放到要打开项目的根目录下，CLion会自动启用ClangFormat并应用设置；
2. 在`文件——设置——编辑器——代码样式`勾选启用`ClangFormat`，然后状态栏会显示`ClangFormat`；
3. 使用快捷键`Ctrl+Alt+L`，可以重新格式化选定代码，没有选定代码时，会格式化整个文件；

# 8 UT的配置

1. 选择`编辑配置--UTTest`，进行配置并应用：
   * 程序参数：`nolog --gtest_filter="xxx"`（后面的--gtest_filter="..."是根据CLion输出信息填写的，过滤掉无关报错）
   * 环境变量：`CHG_HOME=xxx;LD_LIBRARY_PATH=xxx;TZ=BEST-8;BILLING_HOME=xxx`
2. 单个执行配置，在当前界面点击`编辑配置模板`，选择`Google Test`，进行配置并应用：
   * 目标：选择`UTTest`
   * 程序参数：不用填
   * 工作目录：`\usr1\dragon\01_RB\test\rating_charging\cppcode\90_uttest\bin` （需要改成自己的）
   * 环境变量：`CHG_HOME=xxx;LD_LIBRARY_PATH=xxx;TZ=BEST-8;BILLING_HOME=xxx`（需要改成自己的）

# 9 编译

点击运行按钮，如果出现下面的报错：

```sh
c++: internal compiler error
...
```

编译内部错误有可能是后台内存不足导致的，点击左边的构建按钮，可能会编译成功。

如果后面再出现这种错误，就把`构建选项`中的数字改小一点：`-- -j 32`

