---
layout:     post
title:      "手把手教你搭建自己的个人网站"
subtitle:   " \"基于WordPress的个人网站\""
date:       2018-01-25
author:     "flyzy"
header-img: "img/post-bg-2018.jpg"
tags:
    - 建站
    - WordPress
    - 个人博客网站
---

一步一步教你基于WordPress搭建自己的个人博客，WordPress作为成熟的CMS框架，美观，方便，插件多，更新频繁，非常适合个人博客与网站的搭建，适合新手，无需太多的代码基础。

> 原文链接：[手把手教你搭建自己的网站](https://www.flyzy2005.cn/build-page)


购买VPS
-----
- **1. 购买云服务器**

为了搭建个人网站，首先肯定需要一个云服务器。
国内的推荐[腾讯云](https://cloud.tencent.com/redirect.php?redirect=1005&cps_key=42c9b322fd48ff0ce405a0c7d78612fd)，毕竟大公司，工单服务贼及时！还送免费的CDN加速流量~
国外的Vultr是我之前用的服务器，也还不错，全球15个机房中心，采用小时计费策略，需要国外服务器的也可以尝试下~[Vultr购买图解步骤](https://www.flyzy2005.cn/vps/vultr-deploy)

- **2.购买域名**

有了云服务器，还需要一个域名。国内的域名需要备案，购买的话阿里云腾讯云都可以。国外的GoDaddy，name.com也可以试试~但是可能国外的域名DNS解析会比较慢。


搭建Nginx+MySQL+PHP7环境
-----
这一部分介绍如何在Ubuntu上配置Nginx+MySQL+PHP7，针对新手，图解教程，搭建个人网站所需坏境与软件。

- **1.安装Nginx**

Ubutun（本教程是基于Ubuntu 16.04）安装nginx还是很简单的，就两句命令（全部root权限）：
```
apt-get update
 
apt-get install nginx
```
安装好后，可以访问http://xx.xx.xx.xx（或者是你的域名），如果显示下图所示结果，就说明成功了：
![这里写图片描述](http://img.blog.csdn.net/20180125160104206?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2Y2MzI4NTY2OTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- **2.安装MySQL**

还是很简单，一行命令：
```
apt-get install mysql-server
```
输入完之后你会被要求输入root的密码，输完之后就安装成功了~

- **3.安装PHP7**

安装命令：
```
apt-get install php-fpm php-mysql
apt-get install php-fpm php-mysql
```

- **4.配置Nginx使用PHP7**

现在我们已经安装了所有需要的软件，目前要做的是修改Nginx的配置文件来使用PHP processor来处理动态内容。

修改Nginx的配置文件：
```
vim /etc/nginx/sites-available/default
```
添加nginx对PHP的处理，修改后的配置文件如下所示：
![这里写图片描述](http://img.blog.csdn.net/20180125160344568?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2Y2MzI4NTY2OTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

重启Nginx启动新配置文件：
```
/etc/init.d/nginx restart
```
- **5.测试PHP与Nginx有没有集成成功**

添加一个info.php：（这里的 /var/www/html/ 对应配置文件中root的路径）
```
vim /var/www/html/info.php
```
内容为：
```
<php 
phpinfo();
```
访问http://xx.xx.xx.xx（或者是你的域名），如下图所示则说明全部安装成功：
![这里写图片描述](http://img.blog.csdn.net/20180125160606202?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2Y2MzI4NTY2OTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


基于Nginx部署WordPress
-----
这一部分介绍如何在Ubuntu系统下基于Nginx部署搭建WordPress。包括下载WordPress，创建WordPress操作的MySQL数据库和用户，配置WordPress，在Nginx中配置WordPress以及安装WordPress。

- **1.下载WordPress**

直接通过wget命令去它官网下载最新的版本：
```
wget http://wordpress.org/latest.tar.gz
```
解压：
```	
tar -xzvf latest.tar.gz
```

- **2.创建WordPress操作的数据库和用户**

使用第二部分创建MySQL时设置的root密码登录MySQL：
```
mysql -u root -p
```
创建数据库：
```
CREATE DATABASE wordpress;
```
创建用户：
```
CREATE USER wordpress@localhost;
```
设置密码：
```
SET PASSWORD FOR wordpress@localhost=PASSWORD("your password");
```
配置权限：
```
GRANT ALL PRIVILEGES ON wordpress.* TO wordpress@localhost IDENTIFIED BY 'your password';
```
刷新权限配置：
```	
FLUSH PRIVILEGES;
```
退出MySQL：
```	
QUIT;
```

- **3.配置WordPress**

重命名示例文件wp-config（**此处的路径/root/wordpress对应你自己的存放路径**）
```
mv /root/wordpress/wp-config-sample.php /root/wordpress/wp-config.php
```
修改配置文件内容：
修改的内容包括DB_NAME，DB_USER，DB_PASSWORD以及下面的唯一key，其中前三个是在第二步自己设置的内容，唯一key可以直接去它提供的网站上拷贝，修改后的文件如下所示：
![这里写图片描述](http://img.blog.csdn.net/20180125161135803?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2Y2MzI4NTY2OTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- **4.配置Nginx**

将wordpress里面的内容拷贝到Nginx对应root路径下（在第二部分**搭建Nginx+MySQL+PHP7环境**有修改过这个文件）：
```
cp -r /root/wordpress/* /var/www/html
```
修改权限：
```
chown -R www-data:www-data /var/www/html
```
重启Nginx：
```
/etc/init.d/nginx restart
```

- **5.安装WordPress**

全部搞定后，访问你的ip或者是域名应该就是这样子的了：
![这里写图片描述](http://img.blog.csdn.net/20180125161704687?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2Y2MzI4NTY2OTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**选择语言**->**设置标题与管理员用户名与密码以及电子邮件**->**安装WordPress**

安装完后，访问你的IP或者域名，一个初始的博客就搭建好了~：
![这里写图片描述](http://img.blog.csdn.net/20180125161759950?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2Y2MzI4NTY2OTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
访问http://your_ip/wp-login.php，输入刚才设置的账户和密码，进入博客管理界面，在这里可以写文章，改主题，应用插件等等，改完再访问你的博客主页就会看到更新~
![这里写图片描述](http://img.blog.csdn.net/20180125161826205?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2Y2MzI4NTY2OTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

至此，一个基于WordPress的个人博客就已经搭建完毕了~是不是很简单很方便？

更多教程修改WordPress主题，网站升级HTTPS，WordPress搬家&备份等等，请访问原文地址：
[手把手教你搭建自己的个人网站](https://www.flyzy2005.cn/build-page)