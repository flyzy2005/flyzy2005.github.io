---
layout:     post
title:      Ubuntu一键安装Shadowsocks脚本
subtitle:   一键安装shadowsocks&&开启BBR
date:       2018-01-31
author:     flyzy
header-img: "img/post-bg-2018.jpg"
tags:
    - shadowsocks
    - ss
    - 科学上网
    - 一键脚本
---

基于[科学上网：VPS上搭建shadowsocks](https://www.flyzy2005.cn/fan-qiang/shadowsocks/build-shadowsocks-on-vps/)写了一个一键安装shadowsocks的shell脚本。只在Vultr上的Ubunbu 16.04做了测试。内容包括安装shadowsocks+设置shadowsocks开机启动+开启BBR加速。

> 原文链接：[Ubuntu一键安装Shadowsocks脚本](https://www.flyzy2005.cn/fan-qiang/shadowsocks/install-shadowsocks-in-one-command/)

## 服务器购买+连接远程Linux服务器

这里直接根据[Vultr购买图解步骤](https://www.flyzy2005.cn/vps/vultr-deploy/)以及[Xshell连接Linux](https://www.flyzy2005.cn/tech/linux/xshell-connect-linux/)即可。注意这个脚本我只在Vultr的Ubuntu 16.04上做了测试。能力有限，精力有限适配性可能不好，不过如果按照之前的购买步骤选择的Ubuntu 16.04是**绝对可以成功**的。

## 一键安装Shadowsocks
- 1.下载脚本文件
```
git clone https://github.com/Flyzy2005/ss-fly
```
![这里写图片描述](http://img.blog.csdn.net/20180131203806638)

- 2.运行安装代码
```
ss-fly/ss-fly.sh -i password [port]
```
其中**password**换成你要设置的shadowsocks的密码即可，而第二个参数是端口号，可以不加，默认是1024~（可以是ss-fly/ss-fly.sh -i qwerasd，也可以是ss-fly/ss-fly.sh -i qwerasd 8585，后者指定了服务器端口为8585，前者则是默认的端口号1024）：

![这里写图片描述](http://img.blog.csdn.net/20180131203915416)

界面如下就表示安装成功了~：

![这里写图片描述](http://img.blog.csdn.net/20180131203942741)

## 本机上搭建shadowsocks代理实现科学上网
各种客户端版本下载地址：[各版本shadowsocks客户端下载地址](https://www.flyzy2005.cn/fan-qiang/shadowsocks/ss-clients-download/)

以Windows为例：

![这里写图片描述](http://img.blog.csdn.net/20180131204024323)

在状态栏右击shadowsocks，勾选**开机启动**和**启动系统代理**，在**系统代理模式**中选择**PAC模式**，**服务器**->**编辑服务器**，一键安装shadowsocks的脚本默认服务器端口是1024，加密方式是aes-256-cfb，密码是你设置的密码，ip是你自己的VPS ip，保存即可~

**PAC模式**是指国内可以访问的站点直接访问，不能直接访问的再走shadowsocks代理~

OK！搭建shadowsocks完毕！翻墙吧，兄弟！[Google](https://www.flyzy2005.cn/go/go.php?url=https://www.google.com/)

## 开启BBR加速（可选）
BBR是Google开源的一套内核加速算法，可以让你搭建的shadowsocks速度上一个台阶。

- 1.检测Ubuntu内核版本

BBR支持4.9以上的，如果你的版本高于这个则会直接开启BBR加速，如果低于这个版本则会自动下载4.10的并重启：
```
ss-fly/ss-fly.sh -bbr
```

![这里写图片描述](http://img.blog.csdn.net/20180131204235491)

2.开启BBR加速

``` 
ss-fly/ss-fly.sh -bbr
```
![这里写图片描述](http://img.blog.csdn.net/20180131204304862)

如图中红框所示，如果第一步有内核更新，则会自动重启，你需要重新[用Xshell连接你的VPS](https://www.flyzy2005.cn/tech/linux/xshell-connect-linux/)，连接后再执行一次命令则会开启BBR加速~