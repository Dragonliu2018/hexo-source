---
title: Dukou网站自动化签到
tags:
  - selenium
categories:
  - [Python, 爬虫]
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-10 10:47:26
---

# 1 问题需求

登陆Dukou网站，点击签到按钮，每天执行此操作。

# 2 解决

解决方法有两个：

1. 利用selenium编写浏览器爬虫，模拟执行操作；然后利用Windows自带的`任务计划程序`实现定时执行爬虫；
2. 编写ios快捷指令

## 2.1 Windows自动执行

### 2.1.1 爬虫脚本

**参考**：[python用selenium爬取百度搜索结果](https://yuaneuro.cn/archives/3.html)

**dukou_sign_in.py**：

```python
from time import sleep
from selenium import webdriver
from selenium.webdriver.common.by import By


def dukou_sign():
    # 无头浏览器设置
    co = webdriver.ChromeOptions()
    co.headless = False
    browser = webdriver.Chrome(options=co)

    # 登陆
    browser.implicitly_wait(20)  # 如果找不到元素，每隔半秒钟再去界面上查看一次， 直到找到该元素， 或者过了20秒最大时长。
    url = 'https://dukou.icu/user/index'  # 目标网站
    browser.get(url)
    sleep(5)
    username = "xxx"  # dukou用户名
    password = "xxx"  # dukou密码
    browser.find_element(by=By.ID, value="email").send_keys(username)  # 输入框
    browser.find_element(by=By.ID, value="passwd").send_keys(password)  #
    browser.find_element(by=By.XPATH, value='//*[@id="formLogin"]/div[3]/div/div/span/button').click()  # 点击登陆按钮

    # 进行签到
    sleep(5)
    browser.find_element(by=By.XPATH, value='//*[@id="app"]/section/section/main/div/div[2]/div/div/div/div[2]/div[3]'
                                            '/div/div/button[1]').click()  # 点击签到
    # 停止爬虫
    browser.quit()


if __name__ == "__main__":
    dukou_sign()
```

***

selenium打开指定URL后，需要登陆：

<img src="https://s2.loli.net/2022/04/10/bLCpikNS1GY3unM.png" width = "600" height = "400" alt="图片名称" align=center id=127 />

在此页面按F12，找到对应输入框的`id`

<img src="https://s2.loli.net/2022/04/10/toCgMNPVl6ExiJj.png" width = "800" height = "300" alt="图片名称" align=center id=128 />

然后通过ID定位element，并发送对应内容：

```python
browser.find_element(by=By.ID, value="email").send_keys(username)
browser.find_element(by=By.ID, value="passwd").send_keys(password)
```

下一步点击`确定`按钮，通过XPATH定位element，并进行点击：

<img src="https://s2.loli.net/2022/04/10/rhKxLbaFgdlpI6W.png" width = "800" height = "300" alt="图片名称" align=center id=129 />

```python
browser.find_element(by=By.XPATH, value='//*[@id="formLogin"]/div[3]/div/div/span/button').click()
```

登陆后，点击`签到`按钮与上面点击`确定`按钮类似：

```python
browser.find_element(by=By.XPATH, value='//*[@id="app"]/section/section/main/div/div[2]/div/div/div/div[2]/div[3]'
                                            '/div/div/button[1]').click()  # 点击签到
```

### 2.1.2 消息通知脚本

爬虫脚本执行完，需要消息提醒，提高体验。

通过 **win10toast** 实现在 Win10 系统中发送桌面消息通知，[参考](https://zhuanlan.zhihu.com/p/350300654)

安装 win10toast：

```sh
pip install win10toast
```

代码：

```python
from win10toast import ToastNotifier

toaster = ToastNotifier()
toaster.show_toast("Dukou自动签到",
                   "签到成功！",
                   icon_path="./img/dukou.ico",
                   duration=60*60*6)


if __name__ == "__main__":
    pass
```

但是这种方法发送的消息会在一定的时间内消失（由参数`duration`指定，单位是秒），不会在`管理通知`中保存；如果将参数`duration`改的足够大，消息可以"永存"，但是后台会持续运行该程序，不太理想。

下面的程序解决了上面的问题，参考[how to storage notifications into Action center](https://github.com/jithurjacob/Windows-10-Toast-Notifications/issues/76)

**dukou_notice.py**：

```python
from winrt.windows.ui.notifications import ToastNotificationManager, ToastNotification
import winrt.windows.data.xml.dom as dom

notifier = ToastNotificationManager.create_toast_notifier(r'D:\developer\Python\python399\python.exe')  # python程序路径

title = "Dukou自动签到"
desp = "签到成功！"
tString = """<toast duration='short'><audio src  = 'ms-winsoundevent:Notification.Reminder' loop = 'false' silent = 'false'/><visual><binding template='ToastText02'><text id="1">""" + title + """</text><text id="2">""" + desp + """</text></binding></visual></toast>"""

xDoc = dom.XmlDocument()
xDoc.load_xml(tString)
notifier.show(ToastNotification(xDoc))


if __name__ == "__main__":
    pass
```

**注意**：`dukou_notice.py` 脚本不能与 `dukou_sign_in.py` 脚本合在一个文件内，否则会报错：

```sh
_winrt.init_apartment()
RuntimeError: 无法在设置线程模式后对其加以更改。
```

### 2.1.3 定时执行脚本

使用Windows自带的`任务计划程序`实现定时执行脚本，具体参考 [Window自带的定时自动执行程序](https://blog.csdn.net/Icesteam/article/details/112328159)

1. 在Windows搜索框输入`任务计划程序`找到这个软件并打开；

2. 点击`创建任务`：

   <img src="https://s2.loli.net/2022/04/10/XfGe4wvulnDd8gV.png" width = "800" height = "300" alt="图片名称" align=center id=130 />

3. `常规`部分，填写`名称`，勾选`不管用户是否登录都要运行`以及 `使用最高权限运行` ：

   <img src="https://s2.loli.net/2022/04/10/l7CUeAj9sc3BkTr.png" width = "500" height = "400" alt="图片名称" align=center id=131 />

4. `触发器`部分，选择合适的触发条件：

   <img src="https://s2.loli.net/2022/04/10/l4YJQbhWKopZd2e.png" width = "500" height = "400" alt="图片名称" align=center id=132 />

5. `操作`部分，这里运行的程序或者脚本，最好直接选择个bat来执行（bat名称和路径分开放），执行exe有时候会出错，bat里设置的是要执行的Python脚本（bat与python脚本同目录下）：

   <img src="https://s2.loli.net/2022/04/10/ZadxHV2C8P4bS5L.png" width = "500" height = "400" alt="图片名称" align=center id=133 />

   **dukou.bat**：

   ```bat
   python dukou_sign_in.py
   python dukou_notice.py
   ```

6. `条件`和`设置`部分默认即可，然后点击确定即可，前端界面已经出现了设置的任务：

   <img src="https://s2.loli.net/2022/04/10/i64vxcqGRu7C1Kl.png" width = "1000" height = "100" alt="图片名称" align=center id=134 />

7. 到此已经完成问题需求。

## 2.2 ios快捷指令（未实现）
