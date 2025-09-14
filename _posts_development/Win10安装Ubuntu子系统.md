---
title: Win10安装Ubuntu子系统
date: 2021-10-27 09:44:22
tags: 
  - Ubuntu
categories:
    - [环境与工具]
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

1. 在Windows10 Home版操作系统下安装Linux子系统(WSL2)，用于轻量需求，方便快捷。
2. 在微软商店内安装Linux子系统会默认安装到C盘，根据官方文档指示可以安装到其他盘（未成功）。
3. 自定义Linux子系统，安装到非C盘。

# 1 安装WSL2（未成功）

> * [微软WSL官方安装文档（中文）](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10)
> * [微软WSL官方安装文档（英文）](https://docs.microsoft.com/en-us/windows/wsl/install-manual#downloading-distros)

## 1.1 启用`Windows子系统`功能

在安装子系统之前，启用`Windows子系统`功能，在Windows10的Powershell中输入：

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

## 1.2 检查WSL2依赖是否满足

- For x64 systems: **Version 1903** or higher, with **Build 18362** or higher.
- For ARM64 systems: **Version 2004** or higher, with **Build 19041** or higher.
- Builds lower than 18362 do not support WSL 2. Use the [Windows Update Assistant](https://www.microsoft.com/software-download/windows10) to update your version of Windows.

检查Windows版本：`win+R`，输入`winver`

## 1.3 启用`虚拟机`功能

管理员模式打开Powershell：

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

## 1.4 下载Linux内核更新包

1. 下载最新安装包：[跳转](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)
2. 点击安装上述的安装包

## 1.5 将 WSL 2 设置为默认版本

在Powershell中运行：

```powershell
wsl --set-default-version 2
```

## 1.6 安装Linux发行版

### 1.6.1 微软商店安装（只能安装在C盘）

### 1.6.2 命令行安装（可以安装在其他盘）

1. cmd下，在自定义目录中下载：

   ```sh
   curl.exe -L -o ubuntu-1604.appx https://aka.ms/wsl-ubuntu-2004
   ```

2. 安装：

   ```sh
   Add-AppxPackage .\app_name.appx
   ```

   <font color="red">但是出错：</font>

   ```sh
   PS D:\virtual_machine\WSL\Ubuntu> Add-AppxPackage .\Ubuntu2004.appx
   Add-AppxPackage : 部署失败，原因是 HRESULT: 0x80073CF3, 包无法进行更新、相关性或冲突验证。
   从  (Ubuntu2004.appx)  使用程序包 CanonicalGroupLimited.Ubuntu20.04onWindows_2004.2020.424.0_x64__79rhkp1fndgsc 中的目
   标卷 C: 执行的部署 Add 操作失败，错误为 0x80073CF3。有关诊断应用部署问题的帮助，请参阅 http://go.microsoft.com/fwlink/?
   LinkId=235160。
   注意: 有关其他信息，请在事件日志中查找 [ActivityId] a0925605-afc5-0005-a072-92a0c5afd701，或使用命令行 Get-AppPackageLo
   g -ActivityID a0925605-afc5-0005-a072-92a0c5afd701
   所在位置 行:1 字符: 1
   + Add-AppxPackage .\Ubuntu2004.appx
   + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
       + CategoryInfo          : WriteError: (D:\virtual_mach...Ubuntu2004.appx:String) [Add-AppxPackage], IOException
       + FullyQualifiedErrorId : DeploymentError,Microsoft.Windows.Appx.PackageManager.Commands.AddAppxPackageCommand
   ```

   [问题解决](https://answers.microsoft.com/zh-hans/windows/forum/all/add-appxpackage-0x80073cfd/1303974b-3ea0-4436-8cb5-20675b1338c3)，应该是home版的windows有关，暂时没有解决。

***

# 2 安装自定义系统（成功）

> - [微软WSL官方安装文档（中文）](https://docs.microsoft.com/zh-cn/windows/wsl/use-custom-distro)
>
> 安装后的问题：
>
> 1. `ping baidu.com`反应慢，`ping 8.8.8.8`还可以
> 2. 

## 2.1 从容器中导出 tar

在Ubuntu虚拟机中进行操作：

```sh
# 安装docker
# https://www.runoob.com/docker/ubuntu-docker-install.html
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
# 启动 Docker 服务
sudo service docker start
# 下载指定系统镜像
sudo docker pull registry.cn-hangzhou.aliyuncs.com/pwndocker/pwndocker-nocbtm:1.0
# 查看系统镜像
sudo docker images
# 使用 ubuntu 镜像启动一个容器，参数为以命令行模式进入该容器
sudo docker run -it ImageID /bin/bash
# 启动容器
sudo docker start ContainerID
# 查看容器
sudo docker ps -a
# 进入容器
sudo docker attach ContainerID
# 使用 grep 和 awk 获取 CentOS 容器 ID
dockerContainerID=$(sudo docker container ls -a | grep -i {容器名称} | awk '{print $1}')
# 根据容器 ID 导出 tar 文件：
sudo docker export $dockerContainerID > ubuntu.tar
```

## 2.2 将 tar 文件导入 WSL

在Windows的powershell中操作：

```sh
# 在选定目录下导入 tar 文件： wsl --import <DistroName> <InstallLocation> <InstallTarFile>
wsl --import Ubuntu .\Ubuntu .\ubuntu.tar
# 检查已安装的子系统实例
wsl -l -v
# 启动Ubuntu子系统
wsl -d Ubuntu
# 停止Ubuntu子系统
wsl --terminate Ubuntu #停止
# 卸载Ubuntu子系统
wsl --unregister Ubuntu
```

# 3 安装Terminus终端

1. **Terminus**下载安装，点击`设置`，修改`Profiles`，使得默认标签页打开wsl（Ubuntu）

   <img src="https://i.loli.net/2021/09/26/FewNKTpMnikjZ8b.png" width = "700" height = "400" alt="图片名称" align=center id=33 />

2. 将Tabby(**Terminus**)添加到右键打开：

   ```sh
   @echo off
   set ICON=D:\software\terminus\soft\Tabby\Tabby.exe #改成自己的
   set LABEL=Open terminal here
   set COMMAND=D:\software\terminus\soft\Tabby\Tabby.exe \"%%V\" #改成自己的
   set SLUG=Terminal
   
   reg delete "HKCU\Software\Classes\Directory\Background\shell\%SLUG%" /f
   reg delete "HKCU\Software\Classes\Directory\shell\%SLUG%" /f
   reg delete "HKCU\Software\Classes\Drive\shell\%SLUG%" /f
   reg delete "HKCU\Software\Classes\LibraryFolder\Background\shell\%SLUG%" /f
   
   reg add "HKCU\Software\Classes\Directory\Background\shell\%SLUG%" /f /ve /t REG_SZ /d "%LABEL%"
   
   reg add "HKCU\Software\Classes\Directory\Background\shell\%SLUG%" /f /v "Icon" /t REG_SZ /d "%ICON%"
   reg add "HKCU\Software\Classes\Directory\Background\shell\%SLUG%\command" /f /ve /t REG_SZ /d "%COMMAND%"
   reg add "HKCU\Software\Classes\Directory\shell\%SLUG%" /f /ve /t REG_SZ /d "%LABEL%"
   
   reg add "HKCU\Software\Classes\Directory\shell\%SLUG%" /f /v "Icon" /t REG_SZ /d "%ICON%"
   reg add "HKCU\Software\Classes\Directory\shell\%SLUG%\command" /f /ve /t REG_SZ /d "%COMMAND%"
   reg add "HKCU\Software\Classes\Drive\shell\%SLUG%" /f /ve /t REG_SZ /d "%LABEL%"
   
   reg add "HKCU\Software\Classes\Drive\shell\%SLUG%" /f /v "Icon" /t REG_SZ /d "%ICON%"
   reg add "HKCU\Software\Classes\Drive\shell\%SLUG%\command" /f /ve /t REG_SZ /d "%COMMAND%"
   reg add "HKCU\Software\Classes\LibraryFolder\Background\shell\%SLUG%" /f /ve /t REG_SZ /d "%LABEL%"
   
   reg add "HKCU\Software\Classes\LibraryFolder\Background\shell\%SLUG%" /f /v "Icon" /t REG_SZ /d "%ICON%"
   reg add "HKCU\Software\Classes\LibraryFolder\Background\shell\%SLUG%\command" /f /ve /t REG_SZ /d "%COMMAND%"
   ```

# 4 配置oh-my-zsh

zsh是一个Linux下强大的shell, 由于大多数Linux产品安装以及默认使用bash shell, 但是丝毫不影响极客们对zsh的热衷, 几乎每一款Linux产品都包含有zsh，通常可以用apt-get、urpmi或yum等包管理器进行安装。

zsh是bash的增强版，其实zsh和bash是两个不同的概念，zsh更加强大。

通常zsh配置起来非常麻烦，且相当的复杂，所以oh-my-zsh是为了简化zsh的配置而开发的，因此oh-my-zsh算是zsh的配置。

## 4.1 安装zsh

1. 安装zsh

   ```sh
   apt install zsh -y
   ```

2. 检查安装是否成功：

   ```sh
   cat /etc/shells
   # /etc/shells: valid login shells
   /bin/sh
   /bin/bash
   /bin/rbash
   /bin/dash
   /usr/bin/tmux
   /bin/zsh
   /usr/bin/zsh
   ```

3. zsh设为默认shell，新开标签页生效

   ```sh
   chsh -s /bin/zsh
   ```

## 4.2 oh-my-zsh安装与配置

> * [Ubuntu | 安装oh-my-zsh](https://www.jianshu.com/p/ba782b57ae96)

1. 安装：

   ```sh
   sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
   # 或者
   sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
   ```

   出现安装报错：

   ```sh
   curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused
   ```

   **解决**：

   * 如果科学上网应该不会出现此情况；

   * 如果不能科学上网可以这样做：

     * 打开网站: `https://www.ipaddress.com/`， 查询一下 `raw.githubusercontent.com`对应的IP 地址

     * 在hosts文件中添加：`sudo vim /etc/hosts`

       ```sh
       199.232.28.133 raw.githubusercontent.com
       ```

2. 主题使用robbyrussell，`vim ~/.zshrc`

   <img src="https://i.loli.net/2021/09/26/CyOAqiXPmTRYFvz.png" width = "700" height = "300" alt="图片名称" align=center id=34 />

3. 安装插件：

   ```sh
   # 修改配置文件 
   vim ~/.zshrc
   ## 添加内容
   plugins=(git zsh-autosuggestions zsh-syntax-highlighting autojump extract)
   # 解压插件，解压一个命令x全搞定
   extract # 不用安装
   # 安装autojump自动跳转插件
   apt install autojump
   # 安装zsh-autosuggestions语法历史记录插件
   git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
   # 安装zsh-syntax-highlighting语法高亮插件
   git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
   # 配置文件生效
   source ~/.zshrc
   ```

