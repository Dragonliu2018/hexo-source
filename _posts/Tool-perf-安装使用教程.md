---
title: '[Tool] perf 安装使用教程'
categories:
  - [Tool,调优]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-26 00:24:37
tags:
---

# 1 ubuntu 安装步骤

1. 安装`linux-tools-common`：
    
    ```bash
    sudo apt-get install linux-tools-common
    ```
    
2. 此时运行perf仍然失败：
    
    ```bash
    ➜  matrixdb git:(main) ✗ perf
    WARNING: perf not found for kernel 5.15.0-69
    
      You may need to install the following packages for this specific kernel:
        linux-tools-5.15.0-69-generic
        linux-cloud-tools-5.15.0-69-generic
    
      You may also want to install one of the following packages to keep up to date:
        linux-tools-generic
        linux-cloud-tools-generic
    ```
    
3. 根据提示先进行对应包的更新：
    
    ```bash
    sudo apt-get install linux-tools-generic linux-cloud-tools-generic
    ```
    
4. 下载对应包：
    
    ```bash
    sudo apt-get install linux-tools-5.15.0-69-generic linux-cloud-tools-5.15.0-69-generic
    ```
    
5. 进行验证，下载成功：
    
    ```bash
    ➜  matrixdb git:(main) ✗ perf -v
    perf version 5.15.87
    ```
    

# 2 卸载

```bash
sudo apt-get remove linux-tools-common linux-tools-$(uname -r) linux-cloud-tools-$(uname -r)
```

# 3 报错

## 3.1 报错1：The cpu_core/cycles/ event is not supported.

```bash
➜  build git:(p1) ✗ sudo perf record ./test/buffer_pool_manager_instance_test
Error:
The cpu_core/cycles/ event is not supported.
```

在公司 macbook 上没有出现报错，在实验室 win11 上出现上述问题。

**参考** [虚拟机Linux使用perf stat提示cycles not supported](https://www.cnblogs.com/azureology/p/13913540.html)，解决方法如下：

1. 关闭VMware虚拟机电源，找到硬件配置选项中CPU
2. 勾选`☑️虚拟化CPU性能计数器`
3. 重启问题解决（但是我的机器勾选后无法启动虚拟机了🥦

# 4 使用

## 4.1 抓取性能信息

```bash
# 进程
sudo perf record --call-graph=dwarf -p 479061

# 可执行文件
sudo perf record --call-graph=dwarf ./exec
```

## 4.2 结果展示

```bash
sudo perf report -i perf.data --no-inline -g
```

## 4.3 快捷键

1. 展开：`shift` + `+`
2. 

# X 参考

- [**基于Ubuntu 18.04 安装perf工具**](https://www.jianshu.com/p/4659063b3c0d)
- [**How To Install perf-tools-unstable on Ubuntu 22.04**](https://installati.one/install-perf-tools-unstable-ubuntu-22-04/)