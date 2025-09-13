---
title: Cuckoo Sandbox 安装教程
tags:
categories:
  - [环境与工具]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-01-30 21:56:31
---

# 0 简介

Cuckoo Sandbox 是一个开源的自动分析可疑文件的软件。它通过在独立环境内使用自定义组件来监控恶意进程的行为。

它可以分析出以下几种类型的结果:

> - 跟踪恶意软件产生函数调用.
> - 恶意软件执行期间的文件操作，包括新建，删除以及文件下载.
> - 恶意软件的内存转储.
> - PCAP格式的网络流量捕获.
> - 恶意软件运行时的截屏.
> - 虚拟机的完整内存转储文件.

Cuckoo 由一个宿主机（管理组件）加上多个沙箱（物理机或者虚拟机）组成。 宿主机上的管理组件负责了一个样本分析的全部过程，样本的执行过程都是在沙箱中进行。

<img src="https://s2.loli.net/2022/01/30/laJ2s3Nh5oUg8IM.png" width = "500" height = "300" alt="图片名称" align=center id=37 />

* [官网](https://cuckoosandbox.org/)
* [官方文档-英文](https://cuckoo.sh/docs/)
* [官方文档-中文](https://cuckoo-sandbox.readthedocs.io/zh_CN/latest/index.html)

# 1 环境准备

本次搭建Cuckoo环境，主要参考[教程](https://blog.csdn.net/ElsonHY/article/details/115607255)，使用一个宿主机+一个沙箱(guest)的模式，具体如下：

|     软件      |              host              |                guest1                 |             guest2              |
| :-----------: | :----------------------------: | :-----------------------------------: | :-----------------------------: |
| VMware 16 Pro | Ubuntu20.04 LTS(推荐GNU/Linux) | VM window7-64bit(推荐64位Win7或WinXP) | VM Ubuntu虚拟机 virtualbox win7 |

其中Win7需要关闭UAC(User Access Control)。

guest打算使用VMWare下直接虚拟机——win7，但是失败了；

成功实现的guest是Ubuntu下的virtualbox下的win7。

# 2 host安装

## 2.1 创建 Ubuntu20.04 LTS 虚拟机

网上好多参考教程

<font color=red>注意:</font>

* Ubuntu20.04虚拟机需要至少4GB的RAM和60–80GB的硬盘，因为你将在里面运行一个windows7虚拟机
* 请确保不要将您的用户名称为“cuckoo”，否则这可能会与以后的安装冲突。
* 确保您的虚拟机已启用VT-x功能

Vmware中虚拟机启用VT-x功能后，再打开Ubuntu虚拟机出现报错`此平台不支持虚拟化的 AMD-V/RVI。`

解决：关闭 `Windows 功能` 里的 `虚拟机平台`，重启后即可。

## 2.2 依赖

### 2.2.1 安装 Python 库 (Ubuntu/Debian-based)

Cuckoo的管理组件完全由Python脚本编写，所以就需要适合的Python版本。 当前，我们完全兼容的Python版本是 **2.7**。

老版本的Python和Python 3（未来可能会支持） 目前都是不支持的。

***

**依赖库：**<font color=red>(注意：Ubuntu20中已将python2移除)</font>

```sh
# step1 安装python2
sudo apt install python2

# step2 安装pip2，Ubuntu 20已经无法通过apt来安装python2的pip2了
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
sudo python2 get-pip.py
# 使用：python2 -m pip install numpy

# step3 安装Virtualenv
sudo apt-get install virtualenv
# 使用： virtualenv --python=python2 env

# step4 安装其他的
sudo apt install python-dev libffi-dev libssl-dev
sudo apt-get install libjpeg-dev zlib1g-dev swig
sudo apt install python-setuptools 
```

***

如果要使用我们基于Django开发的Web界面, 则MongoDB是必须要安装的：

```sh
sudo apt-get install mongodb
```

***

如果要使用PostgreSQL数据库(推荐), PostgreSQL也必须安装：

```sh
sudo apt-get install postgresql libpq-dev
```

***

如果使用KVM的话，则需要安装KVM相关依赖：

```sh
sudo apt-get install qemu-kvm
sudo apt-get install libvirt-dev libvirt-daemon libvirt-clients
# 未完成
libvirt-bin ubuntu-vm-builder bridge-utils python-libvirt
```

安装Yara：（可选）

```sh
sudo -H pip install yara-python==3.6.3
```

安装ssdeep：

```sh
sudo apt-get install -y ssdeep
```

安装pydeep：(可选，未完成)

```sh
sudo pip install -U pip setuptools
```

安装openpyxl：

```sh
sudo -H pip install openpyxl
```

安装ujson:

```sh
sudo -H pip install ujson
```

现在安装jupyter：

```sh
sudo -H pip install jupyter
```

### 2.2.2 虚拟化软件

Cuckoo沙箱支持大部分的虚拟化软件，可以很方便的添加和使用各种虚拟化支持。

本文档以VirtualBox为例，选择哪种虚拟机软件并不影响后续的分析。

安装VirtualBox：

```sh
sudo apt-get install virtualbox
```

### 2.2.3 安装 tcpdump

Tcpdump用于抓取恶意软件运行过程中产生的所有流量。

安装命令：

```sh
sudo apt-get install tcpdump apparmor-utils
sudo aa-disable /usr/sbin/tcpdump
```

`AppArmor` 只有当PCAP文件生成没有权限的时候才需要，可以参考 [Permission denied for tcpdump](https://cuckoo-sandbox.readthedocs.io/zh_CN/latest/faq/index.html#tcpdump-permission-denied)

Tcpdump需要root权限，如果不想运行在root用户下，需要做以下设置：

```sh
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
```

可以用以下命令验证是否配置正确：

```sh
$ getcap /usr/sbin/tcpdump
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+eip
```

如果没有`setcap`命令， 则需要安装下面的包:

```shell
$ sudo apt-get install libcap2-bin
```

或者 (**不推荐**)

```sh
$ sudo chmod +s /usr/sbin/tcpdump
```

需要注意的是 setcap 命令不安全，有可能造成提权漏洞，我们建议将Cuckoo安装在专有的环境里。

### 2.2.4 安装 Volatility

Volatility 用于分析内存转储文件的可选工具. Cuckoo与Volatility配合，可以更深度和全面的分析，可以防止恶意软件利用rookit技术逃逸沙箱的监控。

为了能够工作正常，Cuckoo要求Volatility版本不低于 2.3， 推荐最新版本2.5。 可以从官网下载 [official repository](https://github.com/volatilityfoundation).

可以查阅Volatility官方文档的安装说明.

```sh
git clone https://github.com/volatilityfoundation/volatility.git 
cd volatility
sudo python setup.py build
sudo python setup.py install
```

安装Distorm3：

```sh
sudo -H pip install distorm3==3.4.4
```

### 2.2.5 安装 M2Crypto

当前 `M2Crypto` 库需要 [SWIG](http://www.swig.org/) 支持. Ubuntu/Debian-like 系统下可以通过以下命令安装:

```sh
sudo apt-get install swig
```

`SWIG` 安装好之后，通过以下命令安装 `M2Crypto`:

```sh
sudo python2 -m pip install m2crypto
```

### 2.2.6 安装 guacd(未完成)

`guacd` 是RDP，SSH，VNC等远程控制的代理层， 是Cuckoo的Web界面的远程终端中使用，可选。

没有它，远程控制功能就无法使用，版本要求0.9.9及以上。我们推荐安装最新版本 使用如下命令安装:

```sh
sudo apt install libguac-client-rdp0 libguac-client-vnc0 libguac-client-ssh0 guacd
```

## 2.3 Cuckoo 安装

### 2.3.1 创建用户（没使用，使用dragon用户下）

Cuckoo可以运行在已有用户下面，也可以新建一个用户来跑Cuckoo。 但是要保证虚拟机和Cuckoo运行在相同的用户下。（让一个单独的用户来运行沙盒是一种很好的安全习惯。）

创建新用户：（密码为root，其他回车即可）

```sh
sudo adduser cuckoo
```

若使用virtualBox则创建一个cuckoo用户，将其添加到vboxusers组里（VirtualBox安装后创建）。

```sh
sudo usermod -a -G vboxusers cuckoo
```

将该用户添加到sudo组。输入：

```sh
sudo adduser cuckoo sudo
```

如果使用KVM，要将用户加入到Libvirtd用户组:

```sh
sudo usermod -a -G libvirtd cuckoo
```

### 2.3.2 安装 Cuckoo

全局安装Cuckoo是没有问题的，但是 **强力推荐** 用 `virtualenv` 来安装：

```sh
$ virtualenv --python=python2 venv
$ . venv/bin/activate
(venv)$ pip install -U pip setuptools
(venv)$ pip install -U cuckoo
```

<font color=green>提醒：cuckoo下载较慢，30min左右</font>

为什么推荐使用 `virtualenv` 呢:

- Cuckoo的依赖并不是用的最新版本，可能会与系统已有的版本冲突.
- 系统中其他软件的安装，可能会导致Cuckoo的依赖产生问题.
- 使用virtualenv，可以让非root用户也可以安装相关软件.
- 简单来说virtualenv是最佳实践.

## 2.4 Cuckoo 工作目录

```sh
# 切换用户cuckoo
su cuckoo
# 进入虚拟环境
. ./venv/bin/activate
# 首次运行
cuckoo -d
```

出现以下界面：

```sh
/home/dragon/graduation/venv/lib/python2.7/site-packages/sflock/decode/office.py:12: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.hazmat.backends import default_backend

    sSSs   .S       S.     sSSs   .S    S.     sSSs_sSSs      sSSs_sSSs
   d%%SP  .SS       SS.   d%%SP  .SS    SS.   d%%SP~YS%%b    d%%SP~YS%%b
  d%S'    S%S       S%S  d%S'    S%S    S&S  d%S'     `S%b  d%S'     `S%b
  S%S     S%S       S%S  S%S     S%S    d*S  S%S       S%S  S%S       S%S
  S&S     S&S       S&S  S&S     S&S   .S*S  S&S       S&S  S&S       S&S
  S&S     S&S       S&S  S&S     S&S_sdSSS   S&S       S&S  S&S       S&S
  S&S     S&S       S&S  S&S     S&S~YSSY%b  S&S       S&S  S&S       S&S
  S&S     S&S       S&S  S&S     S&S    `S%  S&S       S&S  S&S       S&S
  S*b     S*b       d*S  S*b     S*S     S%  S*b       d*S  S*b       d*S
  S*S.    S*S.     .S*S  S*S.    S*S     S&  S*S.     .S*S  S*S.     .S*S
   SSSbs   SSSbs_sdSSS    SSSbs  S*S     S&   SSSbs_sdSSS    SSSbs_sdSSS
    YSSP    YSSP~YSSY      YSSP  S*S     SS    YSSP~YSSY      YSSP~YSSY
                                 SP
                                 Y

 Cuckoo Sandbox 2.0.7
 www.cuckoosandbox.org
 Copyright (c) 2010-2018

=======================================================================
    Welcome to Cuckoo Sandbox, this appears to be your first run!
    We will now set you up with our default configuration.
    You will be able to see and modify the Cuckoo configuration,
    Yara rules, Cuckoo Signatures, and much more to your likings
    by exploring the /home/cuckoo/.cuckoo directory.

    Among other configurable items of most interest is the
    new location for your Cuckoo configuration:
              /home/cuckoo/.cuckoo/conf
=======================================================================

Cuckoo has finished setting up the default configuration.
Please modify the default settings where required and
start Cuckoo again (by running `cuckoo` or `cuckoo -d`).
```

# 3 guest安装(在Ubuntu20.04中的VirtualBox中)

## 3.1 设置virtualbox环境

安装网络工具：`sudo apt install -y net-tools`，现在键入`ifconfig`以查找虚拟机的IP地址信息。

<img src="https://s2.loli.net/2022/01/31/zCv8PyLSDpwJUKO.png" width = "600" height = "300" alt="图片名称" align=center id=39 />

创建一个只支持主机的网络适配器：`vboxmanage hostonlyif create`

为虚拟接口设置IP地址：`vboxmanage hostonlyif ipconfig vboxnet0 --ip 192.168.56.1`

再次运行`ifconfig`，将看到刚刚设置的接口和IP地址：

<img src="https://s2.loli.net/2022/01/31/2uWvZkIcfxSatzw.png" width = "600" height = "300" alt="图片名称" align=center id=41 />

我们要使这些更改在重新启动后仍然有效，并在系统启动期间自动设置。所以我们需要建立一个新的目录：

```sh
sudo mkdir /opt/systemd/
```

然后：

```sh
sudo vim /opt/systemd/vboxhostonly
```

并将以下代码复制到文件中：

```sh
!/bin/bash
hostonlyif create
vboxmanage hostonlyif ipconfig vboxnet0 --ip 192.168.56.1
```

现在我们需要使文件可执行：

```sh
cd /opt/systemd/
sudo chmod a+x vboxhostonly
```

现在我们需要创建一个服务：

```sh
sudo touch /etc/systemd/system/vboxhostonlynic.service
```

编辑文件：

```sh
sudo vim /etc/systemd/system/vboxhostonlynic.service
```

然后：

```sh
Description=Setup VirtualBox Hostonly Adapter
After=vboxdrv.service
[Service]
Type=oneshot
ExecStart=/opt/systemd/vboxhostonly
[Install]
WantedBy=multi-user.target
```

现在安装服务并确保它在引导时加载:

```sh
systemctl daemon-reload
systemctl enable vboxhostonlynic.service
```

网络部分完成。

## 3.2 创建Win7-x64虚拟机

选择镜像 [win7-x64-sp1](https://msdn.itellyou.cn/) 

启动Oracle VirtualBox Manager

单击New图标创建一个新的虚拟机`guest1`，选择合适的操作系统：

<img src="https://s2.loli.net/2022/01/31/VLlkqKRCxXSA6ng.png" width = "550" height = "300" alt="图片名称" align=center id=42 />

现在单击下一步。选择要分配的RAM大小。理想情况下，最小值为4GB。我将设置为4096MB（4GB）:

<img src="https://s2.loli.net/2022/01/31/jqlO5M3kPba9XQI.png" width = "550" height = "300" alt="图片名称" align=center id=43 />

单击下一步。通过单击“创建”创建虚拟硬盘：

<img src="https://s2.loli.net/2022/01/31/5L1zeiETygwM62b.png" width = "550" height = "300" alt="图片名称" align=center id=44 />

选择VDI并单击Next，选择“动态分配”，然后单击“下一步”。

选择你的硬盘大小。我已设置为32GB，现在单击“创建”。

现在你需要把Windows7ISO文件装载到你的Ubuntu虚拟机上。

单击“确定”。这将把Windows7.iso安装到虚拟机中。现在单击VirtualBox中的设置，然后转到存储。现在单击CD（空）。在右侧或光盘驱动器上，单击CD旁边的向下箭头，然后选择主机驱动器作为CD选项。

<img src="https://s2.loli.net/2022/01/31/gQFULNs62cT57HO.png" width = "550" height = "300" alt="图片名称" align=center id=45 />

单击“确定”关闭，然后开始启动虚拟机。按照正常的Windows7安装。我把我的Windows7虚拟机叫做guest1。

现在您的Windows7虚拟机已构建，请登录。（卡顿的原因可能是Ubuntu虚拟机磁盘空间不足，[解决方法](http://dragonliu.tk/2022/02/02/VMWare虚拟机Ubuntu20-04-LTS磁盘空间不足的解决方法整理/)）

## 3.3 进行配置

我们将进行一些更改，以使虚拟机故意易受攻击。转到“开始”并键入组。您应该看到“编辑组策略”选项。

<img src="https://s2.loli.net/2022/02/01/1iFaKsx5WJmSNjy.png" width = "550" height = "300" alt="图片名称" align=center id=46 />

展开“计算机配置”>“Windows设置”>“安全设置”>“本地策略”>“安全选项”。向下滚动到用户帐户控制选项。

<img src="https://s2.loli.net/2022/02/01/N6IvV85oZwDY2c7.png" width = "550" height = "300" alt="图片名称" align=center id=47 />

右键单击“用户帐户控制：管理员批准模式中管理员的提升权限提示的行为”，然后选择“属性”。使用下拉菜单选择“不提示，直接提升”，然后单击“确定”。

<img src="https://s2.loli.net/2022/02/01/FVwSXrlKARNyiUx.png" width = "550" height = "300" alt="图片名称" align=center id=48 />

右键单击“用户帐户控制：检测应用程序安装并提示提升”，然后选择属性。选中“禁用”选项，然后单击“确定”。

<img src="https://s2.loli.net/2022/02/01/9ytdwx5N42ojqbs.png" width = "550" height = "300" alt="图片名称" align=center id=49 />

右键单击“用户帐户控制：以管理员批准模式运行所有管理员”，然后选择属性。选中“禁用”选项，然后单击“确定”。

<img src="https://s2.loli.net/2022/02/01/gvr4TaikESyMt3j.png" width = "550" height = "300" alt="图片名称" align=center id=50 />



现在我们关闭windows防火墙和自动更新，原因为他们可能影响恶意软件的行为。

<img src="https://s2.loli.net/2022/02/01/spFXaOW65ntdRCG.png" width = "550" height = "300" alt="图片名称" align=center id=51 />



<img src="https://s2.loli.net/2022/02/01/spFXaOW65ntdRCG.png" width = "550" height = "300" alt="图片名称" align=center id=52 />

接下来我们需要禁用Windows Defender，将“自动扫描”和“实时保护”关闭：

<img src="https://s2.loli.net/2022/02/01/pZKmeODJ7wCASx2.png" width = "550" height = "300" alt="图片名称" align=center id=53 />



## 3.4 安装软件

单击VirtualBox窗口中的设备并选择InstallVirtualBox Guest Additions完成安装；

添加一个共享文件夹，使用`Downloads`文件夹：

<img src="https://s2.loli.net/2022/02/01/ToqhEst3VQnkKmb.png" width = "550" height = "300" alt="图片名称" align=center id=54 />

转到Windows中的文件资源管理器并键入`\\vboxsvr\*Downloads` 即可访问Ubuntu中的文件夹。

物理机下载[python2.7](https://www.python.org/ftp/python/2.7.8/python-2.7.8.amd64.msi)，将此安装包放入共享文件夹并从Windows7虚拟机运行，安装的时侯选择 add python to path；

下载[Python Pillow](https://pypi.python.org/packages/2.7/P/Pillow/Pillow-2.5.3.win-amd64-py2.7.exe#md5=33c3a581ff1538b4f79b4651084090c8)，将安装包放入共享文件夹并从Windows 7虚拟机运行。

（未完成）现在安装Adobe Reader、Flash、Java和Microsoft Office（可选）以及您认为可能需要的任何其他应用程序。

（未完成）打开Internet Explorer和Adobe Reader以及任何其他可能有可能干扰cuckoo客户端操作的闪屏的应用程序。

## 3.5 安装客户端

上传`agent.py`文件至win7虚拟机：

```sh
cd /home/cuckoo/.cuckoo/agent
cp agent.py ~/Downloads
```

<img src="https://s2.loli.net/2022/02/01/vDYqUjB2SOiP5r6.png" width = "550" height = "300" alt="图片名称" align=center id=55 />

自启动：复制agent.py文件并将其放置在`C:\Users\*USERNAME*\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`
这将在启动虚拟机时启动agent.py。在Windows7虚拟机上，需要显示隐藏的文件和文件夹才能看到AppData文件夹。
您可以通过进入“文件资源管理器”，然后进入“组织”>“文件夹选项”来完成此操作。

<img src="https://s2.loli.net/2022/02/01/k39m7tVqcN2YoTC.png" width = "400" height = "300" alt="图片名称" align=center id=56 />

<img src="https://s2.loli.net/2022/02/01/54dthCes9iuBKDy.png" width = "500" height = "300" alt="图片名称" align=center id=57 />

一旦您对安装了所有必需的软件感到满意，请重新启动Windows7虚拟机并登录。你应该得到一个空白的命令窗口出现。如果出现网络弹出窗口，请单击“允许”。

<img src="https://s2.loli.net/2022/02/01/k39m7tVqcN2YoTC.png" width = "400" height = "300" alt="图片名称" align=center id=58 />

<img src="https://s2.loli.net/2022/02/01/LkHXv6ydEUP7btW.png" width = "400" height = "300" alt="图片名称" align=center id=59 />

现在可以最小化虚拟机。

## 3.6 创建沙盒环境

现在我们已经创建了Windows7虚拟机，我们需要设置网络来创建沙盒环境。单击VirtualBox管理器，然后单击Windows 7虚拟机的设置。选择网络，然后更改附加到：仅主机适配器，然后在名称中选择vboxnet0。单击“确定”。

<img src="https://s2.loli.net/2022/02/01/a1xFih5vb9jWVew.png" width = "600" height = "300" alt="图片名称" align=center id=59 />

现在你需要让这个持久化。

```sh
sudo apt-get install -y iptables-persistent
```

现在我们需要配置IP转发，以便将internet连接从主机路由到来宾虚拟机。我们将使用iptables设置这些网络转发规则：

```sh
sudo iptables -A FORWARD -o eth0 -i vboxnet0 -s 192.168.56.0/24 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

您可以通过运行`sudo iptables-L`来检查您的规则

现在在内核中启用IP转发，以便将这些设置设置为活动（WWW Internet访问需要）：

```sh
echo 1 | sudo tee -a /proc/sys/net/ipv4/ip_forward
sudo sysctl -w net.ipv4.ip_forward=1
```

现在我们需要确保IP转发在重新启动后启动：

```sh
sudo vim /etc/sysctl.conf
```

把#从`net.ipv4.ip_forward=1`项移除，保存退出。
现在运行`sudo su -`然后`iptables-save > /etc/iptables/rules.v4`

现在我们需要配置Windows7虚拟机的IP地址信息。在中打开网络适配器并设置以下信息：

<img src="https://s2.loli.net/2022/02/01/hcsV7YUqwFgCrv1.png" width = "400" height = "300" alt="图片名称" align=center id=60 />

单击“确定”和“确定”之后，您应该再次获得internet连接。

我们现在要拍摄虚拟机的快照，所以单击Machine>takesnapshot。将其命名为Snapshot 1，然后单击ok。

## 3.7 配置cuckoo服务

我们现在需要在ubuntu20上完成cuckoo服务的配置。所有cuckoo配置文件都位于

```bash
~/.cuckoo/conf
```

或您的主目录

```bash
cd .cuckoo/
```

切换到config目录：

```sh
cd /home/cuckoo/.cuckoo/conf
```

通过

```
sudo vim cuckoo.conf
```

编辑buckoo.conf文件确保设置了以下内容：`machine=virtualbox`
和`memory_dump=yes`
和`resultserver ip=192.168.56.1`，
保存退出即可

通过

```sh
sudo vim auxiliary.conf
```

编辑auxiliary.conf文件确保启用了
`sniffer=yes`

保存退出

通过

```sh
sudo vim virtualbox.conf
```

编辑virtualbox.conf文件，并确保
virtualbox mode=gui
和machines=cuckoo1
（如果需要，将名称更改为您的Windows 7虚拟机名称，label=cuckoo1和platform=Windows和ip=192.168.56.101和snapshot=snapshot 1），
保存退出

通过

```
sudo vim processing.conf
```

编辑processing.conf文件，
确保`memory enabled=yes`，

保存退出

使用

```
sudo vim memory.conf
```

编辑memory.conf文件，
确保`basic guest_profile=Win7SP1x64`，

保存退出

通过

```
sudo vim reporting.conf
```

编辑reporting.conf文件，确保singlefile启用
`report.html enabled=yes`和`mongodb enabled=yes`的创建，

保存退出

## 3.8 启动cuckoo

现在关闭Windows7虚拟机，然后重新启动Ubuntu虚拟机。以<font color=red>**同一用户身份**</font>重新登录（不要以cuckoo用户身份登录）

**dragon**用户下激活虚拟环境：

```sh
➜  graduation . ./venv/bin/activate
(venv) ➜  graduation
```

现在要使用cuckoo你首先需要更新cuckoo的签名，所以打开一个终端并键入

```sh
(venv) ➜  graduation cuckoo community                 
/home/dragon/graduation/venv/lib/python2.7/site-packages/sflock/decode/office.py:12: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.hazmat.backends import default_backend
2022-02-02 20:54:28,277 [cuckoo.apps.apps] INFO: Downloading.. https://github.com/cuckoosandbox/community/archive/master.tar.gz
2022-02-02 20:54:33,956 [cuckoo] INFO: Finished fetching & extracting the community files!
```

现在在终端窗口1中键入`cuckoo`并按回车键。你将得到启动cuckoo，它将坐在等待分析任务。

在第二个终端窗口中键入 `cuckoo web runserver 0.0.0.0:8000`

<img src="https://s2.loli.net/2022/02/02/IOe5Xy8GCtPLVjK.png" width = "800" height = "400" alt="图片名称" align=center id=61 />

现在您可以通过`127.0.0.1:8000`访问cuckoo网络服务器，或者通过浏览器访问本地网络上的虚拟机IP地址。

<img src="https://s2.loli.net/2022/02/02/6FYTpnrKShyx1li.png" width = "800" height = "400" alt="图片名称" align=center id=62 />

然后提交分析即可，生成报告如下：

<img src="https://s2.loli.net/2022/02/02/OxT9wCd6FBkUehX.png" width = "800" height = "400" alt="图片名称" align=center id=63 />

# 4 guest安装(在Vmware中，与Ubuntu并列，未完成，失败)

## 4.1 创建win7-x64虚拟机

* 镜像下载：[MSDN](https://msdn.itellyou.cn/) （需要Service Pack 1，否则vmtools安装报错：

  ```sh
  win7企业版安装vmtool提示无法进行，需要更新到SP1
  ```

  <img src="https://s2.loli.net/2022/01/31/vRTVYMeOB6nWCXi.png" width = "600" height = "300" alt="图片名称" align=center id=38 />

  此[教程](https://blog.csdn.net/kinnisoy/article/details/105299515)无效	

* 按照步骤安装即可

* 安装VMware Tools（如果安装失败：

  ```
  Windows 无法验证此驱动程序软件的发布者
  ```

  使用此[ISO映像文件](http://softwareupdate.vmware.com/cds/vmw-desktop/ws/15.5.0/14665864/windows/packages/tools-windows.tar)安装即可，教程的[评论](https://blog.csdn.net/DCTANT/article/details/117886423)中提及的

## 4.2 依赖

### 4.2.1 安装python

Python 是 Cuckoo 客户端（分析器） 正常工作的必须软件。

1. 官网下载安装 Python2.7 版本；
2. 如果安装的时侯选择 add python to path,则会自动完成环境变量配置；否则需要手动添加

Cuckoo 客户端组件依赖于部分额外的Python 库， 包括: [Python Pillow](https://python-pillow.org/): 截图组件需要用到。下载[安装包](https://pypi.python.org/packages/2.7/P/Pillow/Pillow-2.5.3.win-amd64-py2.7.exe#md5=33c3a581ff1538b4f79b4651084090c8)后双击运行即可。

这些组件不是必须要安装的， 但是不安装的话，分析组件的部分功能就无法正常使用。

### 4.2.2 其他软件

至此，Cuckoo 正常工作所需的软件的已经安装完成了。

不过根据你需要分析的文件类型， 也同时需要安装相应的软件， 例如(谷歌)浏览器，PDF阅读器，Office软件等。 记得要关闭这些软件的检查更新和自动更新。

这些额外的软件是否需要安装，完全取决于你是否所需。 可以阅读 [沙箱](https://cuckoo-sandbox.readthedocs.io/zh_CN/latest/introduction/sandboxing.html) 章节了解更多的信息.

## 4.3 网络配置

### 4.3.1 Windows 设置

在配置底层网络之前，可能需要调整一些windows虚拟机的内部配置。

最重要的事情是关闭控制面板中的 *Windows 防火墙* 和 *自动更新*。 这些都会影响恶意软件的行为，进而影响Cuckoo对这些行为的分析。

### 4.3.2 虚拟网络

现在可以决定虚拟机如何访问互联网或者本地局域网。

以前老的版本中， Cuckoo 虚拟机和宿主机之间的数据是通过共享文件夹进行交互。 从0.4版本起，则通过XMLRPC 协议来交互。

所以，需要配置给虚拟机配置静态IP，配置完成后，通过PING来测试虚拟机与宿主机之间 的通信是否正常。不要使用DHCP， 每次IP都不同的情况下，无法正常通信。

这些配置都要依赖于你的需求和所选的虚拟机软件的特性。

> 警告：虚拟网络报错! 虚拟网络配置是非常重要的部分。大部分Cuckoo遇到的问题都与网络配置有关系。 在你配置完成之后， 尽量用PING和TELNET工具测试是否正常。

推荐使用 *Host-Only* 模式的虚拟网络。

***

**win7虚拟机使用nat模式配置静态IP上网：（未完成）** [链接](https://blog.csdn.net/qq_40657585/article/details/108268552)

查看ubuntu20.04 LTS的ip地址，配置window7客户机的ipv4地址的默认网关和DNS地址为ubuntu的ip地址，如下：

<img src="https://s2.loli.net/2022/01/31/zCv8PyLSDpwJUKO.png" width = "600" height = "300" alt="图片名称" align=center id=39 />

***

使用NAT模式DHCP：

* Ubuntu：192.168.10.128
* Win7：192.168.10.134

***

## 4.3 安装客户端

从0.4版本起， Cuckoo设计了以跨平台的交互客户端，可以在Windows， Android， Linux 和Mac OSX 系统上运行。

只有安装和启动了Cuckoo 客户端， 分析才能工作正常。

客户端的安装和启动是十分简单的。

在 `$CWD/agent/` 目录中，可以找到 `agent.py` 文件。 把文件拷贝到虚拟机中， 然后将脚本启动起来。 客户端会启动一个小型的API服务，用于与宿主机通信。

在Windows系统中， 只要将脚本名称 从 `agent.py` 改为 **agent.pyw** ， 可以在运行的时候不显示终端的窗口。

将脚本拷贝到 启动 目录， 即可实现脚本开机自启动。

```sh
reg add "hklm\software\Microsoft\Windows NT\CurrentVersion\WinLogon" /v DefaultUserName /d <USERNAME> /t REG_SZ /f
reg add "hklm\software\Microsoft\Windows NT\CurrentVersion\WinLogon" /v DefaultPassword /d <PASSWORD> /t REG_SZ /f
reg add "hklm\software\Microsoft\Windows NT\CurrentVersion\WinLogon" /v AutoAdminLogon /d 1 /t REG_SZ /f
reg add "hklm\system\CurrentControlSet\Control\TerminalServer" /v AllowRemoteRPC /d 0x01 /t REG_DWORD /f
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v LocalAccountTokenFilterPolicy /d 0x01 /t REG_DWORD /f


reg add "hklm\software\Microsoft\Windows NT\CurrentVersion\WinLogon" /v DefaultUserName /d guest2 /t REG_SZ /f
reg add "hklm\software\Microsoft\Windows NT\CurrentVersion\WinLogon" /v DefaultPassword /d root /t REG_SZ /f
reg add "hklm\software\Microsoft\Windows NT\CurrentVersion\WinLogon" /v AutoAdminLogon /d 1 /t REG_SZ /f
reg add "hklm\system\CurrentControlSet\Control\TerminalServer" /v AllowRemoteRPC /d 0x01 /t REG_DWORD /f
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v LocalAccountTokenFilterPolicy /d 0x01 /t REG_DWORD /f
```

# 5 参考

* [Cuckoo-Sandbox学习-安装篇](https://fl4g.cn/2018/06/06/Cuckoo-Sandbox%E5%AD%A6%E4%B9%A0-%E5%AE%89%E8%A3%85%E7%AF%87/)
* [Ubuntu20.04系统Cuckoo sandbox所需环境搭建及安装(超详细图文)](https://blog.csdn.net/ElsonHY/article/details/115607255)
