---
title: Pycharm远程开发
tags:
categories:
  - [环境与工具]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-21 22:22:11
---

# 1 引入

最近跑深度学习模型，自己的笔记本吃不消了，于是打算使用实验室的机子。为了提高开发效率，使用PyCharm进行远程开发。

# 2 查看远程环境

1. 使用`MobaXterm`软件登陆实验室的机子，并将代码+数据集上传至指定位置；（其实这一步可以不做，在后面使用PyCharm进行同步）

2. 查看机子的系统：Centos7

   ```sh
   $ cat /etc/os-release
   NAME="CentOS Linux"
   VERSION="7 (Core)"
   ID="centos"
   ID_LIKE="rhel fedora"
   VERSION_ID="7"
   PRETTY_NAME="CentOS Linux 7 (Core)"
   ANSI_COLOR="0;31"
   CPE_NAME="cpe:/o:centos:centos:7"
   HOME_URL="https://www.centos.org/"
   BUG_REPORT_URL="https://bugs.centos.org/"
   
   CENTOS_MANTISBT_PROJECT="CentOS-7"
   CENTOS_MANTISBT_PROJECT_VERSION="7"
   REDHAT_SUPPORT_PRODUCT="centos"
   REDHAT_SUPPORT_PRODUCT_VERSION="7"
   ```

# 3 配置远程Python环境

1. 发现机子是使用conda管理python的，查看系统中的所有环境：

   ```sh
   (base) [ccyin@localhost Malware-Detection]$ conda info -e
   # conda environments:
   #
   base                  *  /home/ccyin/miniconda3
   junxuan                  /home/ccyin/miniconda3/envs/junxuan
   ```

2. 创建环境，名称为`malware`：

   ```sh
   conda create --name malware python=3.9
   # 指定Python版本是3.9（不用管是3.9.x，conda会为我们自动寻找3.9.x中的最新版本）
   ```

3. 激活环境：

   ```sh
   (base) [ccyin@localhost Malware-Detection]$ source activate malware
   (malware) [ccyin@localhost Malware-Detection]$
   ```

   命令行前出现`(malware)`证明成功激活`malware`环境

4. 查看python版本：

   ```sh
   (malware) [ccyin@localhost Malware-Detection]$ python --version
   Python 3.9.12
   ```

5. 安装一些库：

   ```sh
   pip install torch
   pip install torchvision
   pip install d2l
   ```

6. 查看已经安装的库：

   ```sh
   conda list
   ```

# 4 本地代码备份

后期本地与服务器的代码同步，以防万一，把本地代码整到GitHub上。

1. 在项目主目录生成空版本库；

2. 由于数据集占空间较大，不上传云端，所以新建`.gitignore`将一些文件夹进行忽略：

   ```sh
   .idea
   __pycache__
   data
   model
   ```

3. Github上新建仓库，然后将本地代码push上去。

# 5 配置PyCharm

1. 将本地的`Python Interperter`更换为远端的，新建python解释器，选择`SSH Interpreter`，输入IP+用户名+密码：

   <img src="https://s2.loli.net/2022/04/21/mQNWwoLCqUBE3Zi.png" width = "800" height = "500" alt="图片名称" align=center id=191 />

2. 查找conda虚拟环境`malware`中的解释器：（`3 配置远程Python环境`中已经创建好`malware`虚拟环境）

   首先确定虚拟环境`malware`的目录：`/home/ccyin/miniconda3/envs/malware`

   ```sh
   $ conda info -e
   # conda environments:
   #
   base                     /home/ccyin/miniconda3
   malware               *  /home/ccyin/miniconda3/envs/malware
   ```

   然后点开`bin`，选择`python`（**注意是`python`不是`python3`**）

   <img src="https://s2.loli.net/2022/04/21/6VolZ1XPqikx7rB.png" width = "750" height = "500" alt="图片名称" align=center id=192 />

3. 选择好解释器之后，就要确定项目在服务器上的同步路径，默认是在`/tmp/`下的文件夹中，现在修改为之前上传的目录：

   ```sh
   (malware) [ccyin@localhost Malware-Detection]$ pwd
   /home/ccyin/zhenlong/Malware-Detection
   ```

   <img src="https://s2.loli.net/2022/04/21/35wchZX61t7Jqnv.png" width = "750" height = "500" alt="图片名称" align=center id=193 />

4. 点击确定后，然后在`Collectiong files...`和`Uploading xxx`，应该是更新远端文件，因为数据集中有许多文件，所以比较慢；（这样看来，之前上传代码+数据集显得多此一举🥦）

5. 更新完毕后，可以**从右边“remote host”中看到服务器上的文件**，而且标绿了，表示其为我们对应的同步路径：

   <img src="https://s2.loli.net/2022/04/22/Mu5KH72ORaS6oQi.png" width = "550" height = "500" alt="图片名称" align=center id=194 />

6. **要对该文件进行修改时，注意一定要先Download下来**，**不可直接双击点开**。Remote Host里的文件，当你执行非打开操作的时候，所指代的都是服务器上的文件，比如你可以从这里选择下载，进行比对操作等。但是当你从Remote Host双击打开文件时，这时打开的就不是服务器上的，而是一个镜像文件。所以如果直接修改从Remote Host双击打开的文件，修改是无效的。一定要先下载下来，进行修改，然后再上传过去。（**第7步设置自动同步，这样本地的修改会自动上传至远端，但是在远端修改或添加删除文件，不会影响本地**）

   <img src="https://s2.loli.net/2022/04/22/QaDvPkOryYlbmgx.png" width = "450" height = "250" alt="图片名称" align=center id=195 />

7. 确保工具栏中的`Tools/Deployment/Automatic Upload`前面有对勾（默认有），这样以来，在本地的修改会自动同步到服务器上，我不需要手动进行上传。

   <img src="https://s2.loli.net/2022/04/22/lVEc6rSKUWR5LPi.png" width = "450" height = "300" alt="图片名称" align=center id=196 />

8. 此外，如果运行python文件需要使用到命令行参数的时候，可以在`Configurations`中进行增加：

   <img src="https://s2.loli.net/2022/04/22/yYbOl6i5cxI2zXg.png" width = "750" height = "400" alt="图片名称" align=center id=197 />

# 6 后台运行py脚本

训练脚本耗时长，使用pycharm直接运行脚本，可能会因为笔记本息屏、断网等因素而中断，所以考虑将脚本挂在后台运行。

1. 使用`MobaXterm`登陆远程主机；

2. 切换到项目目录并激活虚拟机环境；

3. 后台运行脚本：

   ```sh
   (malware) [ccyin@localhost Malware-Detection]$ nohup python -u train_textcnn.py > textcnn.log 2>&1 &
   [1] 46583
   (malware) [ccyin@localhost Malware-Detection]$ nohup python -u train_rnn.py > rnn.log 2>&1 &
   [2] 46823
   ```

4. 检查下后台是否运行：

   ```sh
   (malware) [ccyin@localhost Malware-Detection]$ ps aux|grep python
   ...
   ccyin     46583 70.4  4.3 67868816 11451892 pts/13 Rl 14:20   1:47 python -u train_textcnn.py
   ccyin     46823  129  0.8 41704128 2243816 pts/13 Rl 14:23   0:09 python -u train_rnn.py
   ```

5. 此时后台成功运行，笔记本断网、关机都不会影响脚本的运行。

# 7 取消后台运行

1. 确定运行脚本的进程ID(`6.3`中已经给出ID)，也可以通过下面的命令获取：

   ```sh
   (malware) [ccyin@localhost Malware-Detection]$ ps aux|grep python
   ...
   ccyin     46583 70.4  4.3 67868816 11451892 pts/13 Rl 14:20   1:47 python -u train_textcnn.py
   ccyin     46823  129  0.8 41704128 2243816 pts/13 Rl 14:23   0:09 python -u train_rnn.py
   ```

2. 杀掉进程：

   ```sh
   $ kill -9 46583
   $ kill -9 46823
   ```

# X 参考

* [Git忽略文件.gitignore的使用](https://www.jianshu.com/p/a09a9b40ad20)
* [Conda使用指南](https://zhuanlan.zhihu.com/p/44398592)
* [PyCharm远程开发的配置与流程](https://zhuanlan.zhihu.com/p/93236936)
* [linux 下后台运行python脚本](https://www.jianshu.com/p/4041c4e6e1b0)
* [nohup后台python3程序及关闭](https://blog.csdn.net/lingyunxianhe/article/details/119328987)
