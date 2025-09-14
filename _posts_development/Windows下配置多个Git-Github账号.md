---
title: Windows下配置多个Git/Github账号
tags:
  - Git
categories:
  - 环境与工具
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-07-05 12:53:24
---

# 1 引入

一般情况下，一台电脑配置一个Git账号。但是当需要两个或以上的Github账号时，来回切换非常麻烦，所以需要配置多个Git账号。

| Github账号 |      邮箱       | Git SSH KEY  |
| :--------: | :-------------: | :----------: |
|   User1    | User1@email.com | id_rsa_User1 |
|   User2    | User2@email.com | id_rsa_User2 |

# 2 生成SSH KEY

1. 打开CMD或者GIT Bash，生成第一个账号的KEY：

   ```sh
   E:\>ssh-keygen -t rsa -f "C:\Users\Dragon Liu\.ssh\id_rsa_User1" -C "User1@email.com"
   Generating public/private rsa key pair.
   Enter passphrase (empty for no passphrase):
   Enter same passphrase again:
   Your identification has been saved in C:\Users\Dragon Liu\.ssh\id_rsa_User1.
   Your public key has been saved in C:\Users\Dragon Liu\.ssh\id_rsa_User1.pub.
   The key fingerprint is:
   SHA256:QtnbRf80ayMhPJvBE1lUHsDvhdLhgSeA7kdofqzLT/Y User1@email.com
   The key's randomart image is:
   +---[RSA 3072]----+
   |         ..o*=oo |
   |       o. oo+o= .|
   |      o... B.*o*.|
   |     .  +o..O =++|
   |      .+So.o o.+o|
   |       .o +   o..|
   |         +o      |
   |       ..o .     |
   |        oo. E    |
   +----[SHA256]-----+
   ```

2. 生成第二个账号的KEY：

   ```sh
   E:\>ssh-keygen -t rsa -f "C:\Users\Dragon Liu\.ssh\id_rsa_User2" -C "User2@email.com"
   Generating public/private rsa key pair.
   Enter passphrase (empty for no passphrase):
   Enter same passphrase again:
   Your identification has been saved in C:\Users\Dragon Liu\.ssh\id_rsa_User2.
   Your public key has been saved in C:\Users\Dragon Liu\.ssh\id_rsa_User2.pub.
   The key fingerprint is:
   SHA256:Ib6+x8PwGBbgMhBg0l/4IGWBCaTF81mY9BIUjrr8DI8 User2@email.com
   The key's randomart image is:
   +---[RSA 3072]----+
   |BB.*O*           |
   |=.B+*oo          |
   |...*oBo .        |
   | .o =oo. .       |
   |.  o  ..S        |
   |..    +.         |
   |.o   ..B         |
   |  *  .. *        |
   | E +  oo .       |
   +----[SHA256]-----+
   ```

3. 进入当前用户的.ssh目录查看，生成id_rsa私钥文件和id_rsa.pub公钥文件，如下截图：

   <img src="https://s2.loli.net/2022/07/05/5Qoa8DRf41pGL2A.png" width = "600" height = "400" alt="图片名称" align=center id=207 />

# 3 添加至Github

1. 用户User1和User2分别登陆Github，点击`settings`，选择`SSH and GPG keys`，点击`New SSH key`：

   <img src="https://s2.loli.net/2022/07/05/bCyVkL3mYHnGteK.png" width = "700" height = "300" alt="图片名称" align=center id=208 />

2. `Title`随便填，`Key`填`id_rsa_xxx.pub`的内容（公钥）：

   <img src="https://s2.loli.net/2022/07/05/1N5dJFMWxTKG3pE.png" width = "600" height = "300" alt="图片名称" align=center id=209 />

# 4 配置config文件

1. 在.ssh目录下创建一个`config`文件，每个账号配置一个Host节点。主要配置项说明：

   ```sh
   Host <主机别名>
       HostName <服务器真实地址>
       IdentityFile <私钥文件路径>
       PreferredAuthentications <认证方式>
       User <用户名>
   ```

2. 配置文件内容：

   ```sh
   # 配置User1
   Host User1.github.com
       HostName github.com
       IdentityFile "C:\\Users\\Dragon Liu\\.ssh\\id_rsa_User1"
       PreferredAuthentications publickey
       User User1
   
   # 配置User2
   Host User2.github.com
       HostName github.com
       IdentityFile "C:\\Users\\Dragon Liu\\.ssh\\id_rsa_User2"
       PreferredAuthentications publickey
       User User2
   ```

3. 配置完成后，在cmd或者Git Bash中输入以下命令测试该用户的SSH密钥是否生效：

   ```sh
   ssh -T git@User1.github.com
   ssh -T git@User2.github.com
   ```

# 5 使用

1. 为各仓库单独配置用户名和邮箱：

   ```sh
   git config user.name "User1"
   git config user.email "User1@email.com"
   ```

2. 远程仓库地址需要修改：

   ```sh
   # 原来的
   git@github.com:xxx/xxxxx.git
   
   # 进行修改
   git remote rm origin
   git remote add origin git@User1.github.com:xxx/xxxxx.git
   ```

# X 参考

* [配置多个Git账号（windows 10）](https://blog.csdn.net/q13554515812/article/details/83506172)
