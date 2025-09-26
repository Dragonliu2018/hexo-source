---
title: '[doris] 编译报错汇总'
categories:
  - [DB,doris]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2025-09-26 09:09:04
tags:
---

# macos 编译报错

## **报错1: mvn is not found**

【**报错**】

```bash
➜  doris git:(master) bash build.sh
Python 3.11.6
The JAVA_HOME environment variable is not defined correctly,
this environment variable is needed to run this program.
Error: mvn is not found
```

***

【**解决**】

设置 JAVA_HOME 环境变量。

https://blog.csdn.net/watson2017/article/details/103243301


## **报错2: 代码 bug**

【**报错**】

```bash
/Users/dragonliu/code/doris/be/src/vec/core/extended_types.h:42:8: error: reference to 'is_signed' is ambiguous
```

***

【**解决**】

代码 bug，新版已经修复：

https://github.com/apache/doris/commit/462db3d5bea5fe3e4e3b9c6e592b91ad2aff6ba4


## **报错3: swapon: command not found**

【**报错**】

```bash
➜  be git:(master) sh bin/start_be.sh --daemon
bin/start_be.sh: line 86: swapon: command not found
```

***

【**解决**】

修改 `doris/output/be/bin/start_be.sh`，将 swapon 部分注释掉

```bash
# if [[ "$(swapon -s | wc -l)" -gt 1 ]]; then
#     echo "Please disable swap memory before installation."
#     exit 1
# fi
```

## **报错4: command not found: mysql → 未解决**

【**报错**】

```bash
➜  bin git:(master) mysql -h 127.0.0.1 -P 9030 -u root
zsh: command not found: mysql
```

***

【**解决**】


下载 mysql client:

```bash
brew install mysql-client
```

因为与 mysql (which contains client libraries) 冲突，需要配置环境变量：

```bash
echo 'export PATH="/usr/local/opt/mysql-client/bin:$PATH"' >> ~/.zshrc
~ source ~/.zshrc
```

错误：

```bash
➜  bin git:(master) mysql -h 127.0.0.1 -P 9030 -u root
ERROR 2003 (HY000): Can't connect to MySQL server on '127.0.0.1:9030' (61)
```