---
layout:     post
title:      "树莓派3代从0到连接WIFI访问（无显示器）"
subtitle:   " \"Hello Raspberry Pi\""
date:       2017-02-07 22:00:00
author:     "Movesan"
header-img: "img/post-bg/post-bg-rpi.jpg"
catalog: true
tags:
    - 树莓派

---

## 前言

Raspberry Pi（中文名为“树莓派”，简写为RPi）是一款信用卡大小的卡片式电脑，是为学生计算机编程教育而设计的。自2012年问世以来，受众多计算机发烧友和创客的追捧，曾经一“派”难求。本文将讲述树莓派从购买到WiFi设置全过程，无论你是否有基础，都能打造属于自己的树莓派！

Here we go!

![img](/img/post-in/post-pi.png)

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

本文为无显示器的树莓派教程，当然如果你想搭配显示器也可以自行选择触摸屏和相关的鼠标键盘（树莓派专用）来打造树莓派。后来经过尝试一般的鼠标键盘也是可以连接树莓派3的，显示器连接只需一根HDMI线就可以了。

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

![img](/img/post-in/post-pi-imager.png)

左侧Image file栏选择刚才下载并解压好的树莓派系统镜像文件（后缀为.img），右侧Device栏选择SD卡，再按下「Write」按钮，过几分钟就完成了

#### 启动Pi

完成SD卡烧录之后，就可以把SD卡插入Pi中，

接下来就可以连接电源开机了（树莓派无开关键，所以接入电源即为开机）

---

## SSH连接Pi

树莓派开机后，在无显示器的情况下，我们还无法正常访问，此时需要一根网线一端连接树莓派，一端连接路由器，然后在电脑中打开浏览器，在你的路由器管理界面查看主机名叫做“raspberrypi”的设备的IP地址，则为目前路由器为树莓派分配的IP。

![img](/img/post-in/post-pi-ip.png)

通过此IP，我们就可以通过SSH协议访问树莓派了（树莓派3代是默认打开SSH服务的），此时我们需要下载SSH客户端进行连接树莓派，Windows系统可以使用[PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。

![img](/img/post-in/post-pi-putty.png)

pi的默认用户名、密码分别是pi、raspberry。

密码输入正确后，可以看到：

![img](/img/post-in/post-pi-ssh.png)


在无法远程访问树莓派的情况下，官方文档还提供了一种方法来手动让SSH开机自启。在向SD卡写入镜像完成后，你会在“此电脑”中看到SD卡的“boot”分区。在这个“boot”分区的根目录下新建一个没有拓展名，名字为的“ssh”空文件，就能开机启用SSH。

---

## 设置WiFi连接

#### 修改/etc/network/interfaces

运行 sudo vim /etc/network/interfaces 命令，打开后作如下修改：

    auto lo eth0 wlan0 wlan1

    iface lo inet loopback
    iface eth0 inet dhcp

    allow-hotplug wlan0 wlan1

    iface wlan0 inet dhcp
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

    iface wlan1 inet manual
    pre-up which owifi
    up owifi start
    down owifi stop

建议，若你不使用树莓派的有线网口连接网络的话，最好把  /etc/network/interfaces  文件第一行（也可能不在第一行）中 auto lo eth0 wlan0 的 eth0 删掉。因为它会导致树莓派开机时等待有线网卡
动态分配IP，但实际上你的有线网口并没有连接到路由器，这里会让内核等待更长的时间，从而拖慢开机速度。

但是要注意，删掉 eth0 的话就意味着无法通过网线连接路由器了，要想连接就必须重新加上 eth0 。第一次设置WiFi连接后接网线一直连不上，因为你可能不在家或者换到其他网络环境，所以想要连接树莓派还需要
通过网线连接才能添加新的WiFi，所以这里要注意下。


#### 修改/etc/wpa_supplicant/wpa_supplicant.conf

除  /etc/network/interfaces  之外，你还需要修改  /etc/wpa_supplicant/wpa_supplicant.conf。

运行 sudo vim /etc/wpa_supplicant/wpa_supplicant.conf 命令，
打开  /etc/wpa_supplicant/wpa_supplicant.conf  照着下面的样子添加（请不要删除原先就已经存在的任何行）：

    # 最常用的配置。WPA-PSK 加密方式。
    network={
    ssid="WiFi-name1"
    psk="WiFi-password1"
    priority=5
    }

    network={
    ssid="WiFi-name2"
    psk="WiFi-password2"
    priority=4
    }

priority 是指连接优先级，数字越大优先级越高（不可以是负数）。

按照自己的实际情况，修改这个文件。

例如，你家中有3个WiFi，分别为WiFi-A、WiFi-B和WiFi-C。你希望树莓派的连接优先级为 WiFi-A>WiFi-B>WiFi-C，则整个配置文件看起来像这样：

    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1

    network={
        ssid="WiFi-A"
        psk="12345678"
        priority=5
    }

    network={
        ssid="WiFi-B"
        psk="12345678"
        priority=4
    }

    network={
        ssid="WiFi-C"
        psk="12345678"
        priority=3
    }

到这里WiFi配置就结束了，重启树莓派后，树莓派将自动连接到WiFi，你可以不用网线直接从路由器管理页面找到对应的IP，然后通过SHH访问树莓派。当然通过这种方式树莓派的IP是动态分配的，我们需要设置静态IP，如果想要更方便还可以给树莓派设置域名，通过域名访问树莓派。

---

## 引用链接

[[linux]树莓派入手体验和系统安装 - Ron Ngai](http://www.cnblogs.com/rond/p/4970071.html)<br>
[RASPBERRY PI 3 MODEL B系統安裝指南 - skynet](http://hophd.com/raspberry-pi-3-model-b-installation/)<br>
[Raspberry Pi 3 无显示器 安装指南 - xusiwei's blog](https://xusiwei.github.io/post/2016/raspberry-pi-headless-setup/)<br>
[树莓派连接WiFi（最稳定的方法） - cmgine](https://i.cmgine.net/archives/11053.html)<br>
