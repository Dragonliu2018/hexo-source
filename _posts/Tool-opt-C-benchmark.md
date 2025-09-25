---
title: '[Tool][opt] C++ benchmark'
categories:
  - [Tool,opt]
  - [PL,C++]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-26 00:30:37
tags:
---

# 1 编译

1. clone 仓库：`git clone https://github.com/google/benchmark.git`
2. 新建 build：
    
    ```bash
    cd benchmark
    mkdir build
    cd build
    ```
    
3. cmake & 下载依赖:
    
    ```bash
    cmake -DBENCHMARK_DOWNLOAD_DEPENDENCIES=on -DCMAKE_BUILD_TYPE=Release ../
    ```
    
4. 编译：
    
    ```bash
    cmake --build "build" --config Release
    ```
    

# 2 编写测试

# 3 在线 benchmark

https://quick-bench.com/q/6tDxsmk3FMX55B8W1RrdiG_s7_k

# X 参考
