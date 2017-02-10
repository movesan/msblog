---
layout:     post
title:      "树莓派3代从0到连接WIFI访问（无显示器）"
subtitle:   " \"Hello Raspberry Pi\""
date:       2017-02-07 22:00:00
author:     "Movesan"
header-img: "img/in-post/post-bg-rpi.jpg"
catalog: true
tags:
    - 树莓派

---

## 前言

Raspberry Pi（中文名为“树莓派”，简写为RPi）是一款信用卡大小的卡片式电脑，是为学生计算机编程教育而设计的。自2012年问世以来，受众多计算机发烧友和创客的追捧，曾经一“派”难求。本文将讲述树莓派从购买到WiFi设置全过程，无论你是否有基础，都能打造属于自己的树莓派！

Here we go!

![img](/img/in-post/post-pi.png)

---

## 树莓派的购买

树莓派正常启动需要主板+电源+TF卡，一个主板是启动不了的，那么如何购买树莓派呢以及都需要哪些组件？
目前树莓派比较常见的有2代和3代，建议购买3代，搭建树莓派，你需要有以下设备：

必须内容：

> 树莓派主板  <br>
> 5V 2.5A电源  <br>
> 电源线  <br>
> SD卡  <br>

以下非必需：

> 散热片（可选，建议购买）  <br>
> 风扇（可选，建议购买）  <br>
> 外壳（可选，建议购买）  <br>
> 收纳盒（可选）

其中必须具备的为主板、电源、电源线、SD卡；树莓派本身主板是没有类似电脑中的硬盘系统的，SD卡则代替作为了树莓派的硬盘，操作系统是装到SD卡上的；

对于树莓派的购买可在某宝某东进行购买，树莓派分为国产和英国原产，其实没啥区别，只是英国原产相对于贵那么十几块二十块，网上的树莓派卖家基本都有树莓派套餐，包括了上述所列的内容，建议购买套餐，价格在200-400元不等，另外对于电源线的购买最好选择带电源开关的。

SD卡不建议购买装好树莓派系统的，因为不能保证系统为最新系统，最好买空白卡进行自己安装。

本文为无显示器的树莓派教程，当然如果你想搭配显示器也可以自行选择触摸屏和相关的鼠标键盘（树莓派专用）来打造树莓派。

---

## 系统安装

#### 下载系统镜像

官网的[下载页面](https://www.raspberrypi.org/downloads/)可以找到pi的系统镜像。

官方首推新手使用的是NOOBS和raspbian，而NOOBS和Raspbian都提供了完整版和Lite版。

NOOBS的完整版是包含了Raspbian，可以离线安装raspbian。Lite版仅有一个系统安装程序，需要连接网络才能安装具体的操作系统。

Raspbian的完整版是一个完整的桌面镜像，Lite版预装的软件包要少一些（联网后可以继续安装）。

我下载的是[RASPBIAN JESSIE WITH PIXEL](https://downloads.raspberrypi.org/raspbian_latest)

下载完成后解压zip文件，将会得到一个img文件。

#### 将镜像写入SD卡

Windows系统上可以用 [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/) 进行写入。

将SD卡插入电脑，然后执行Win 32 Disk Imager，如下：

![img](/img/in-post/post-pi-imager.png)

左侧Image file栏选择刚才下载并解压好的树莓派系统镜像文件（后缀为.img），右侧Device栏选择SD卡，再按下「Write」按钮，过几分钟就完成了

#### 启动Pi

完成SD卡烧录之后，就可以把SD卡插入Pi中，

接下来就可以连接电源开机了（树莓派无开关键，所以接入电源即为开机）

---

## SSH连接Pi

树莓派开机后，在无显示器的情况下，我们还无法正常访问，此时需要一根网线一端连接树莓派，一端连接路由器，然后在电脑中打开浏览器，在你的路由器管理界面查看主机名叫做“raspberrypi”的设备的IP地址，则为目前路由器为树莓派分配的IP。

通过此IP，我们就可以通过SSH协议访问树莓派了（树莓派3代是默认打开SSH服务的），此时我们需要下载SSH客户端进行连接树莓派，Windows系统可以使用[PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。

pi的默认用户名、密码分别是pi、raspberry。

密码输入正确后，可以看到：


在无法远程访问树莓派的情况下，官方文档还提供了一种方法来手动让SSH开机自启。在向SD卡写入镜像完成后，你会在“此电脑”中看到SD卡的“boot”分区。在这个“boot”分区的根目录下新建一个没有拓展名，名字为的“ssh”空文件，就能开机启用SSH。

---

## 设置WiFi连接

---

## 后续

---

## 引用链接

[http://www.cnblogs.com/rond/p/4970071.html](http://www.cnblogs.com/rond/p/4970071.html)<br>
[http://hophd.com/raspberry-pi-3-model-b-installation/](http://hophd.com/raspberry-pi-3-model-b-installation/)<br>
[https://xusiwei.github.io/post/2016/raspberry-pi-headless-setup/](https://xusiwei.github.io/post/2016/raspberry-pi-headless-setup/)<br>
