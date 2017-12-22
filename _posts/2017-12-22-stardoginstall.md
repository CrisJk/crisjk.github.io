---
layout: post
author: Kuang
title: RDF数据库Stardog安装(Ubuntu)
categories: knowledgeGraph
tags: stardog
---

RDF是资源描述框架(Resource Description Framework)的简称，RDF是一种用于描述网络资源的标准。很多知识图谱把数据存储成RDF格式，使用RDF数据库来管理这些数据十分有必要。这里记录一下一种功能强大的RDF数据库Stardog的安装。




## 获得download link和license key

Stardog提供企业版和社区版，企业版有30天试用期，社区版当然是免费的。这里我只需要使用到社区版的功能。在其[主页](http://www.stardog.com/)上点击Try Stardog按钮，填写好信息后，stardog会给你发一个邮件，邮件包含download link和license key。由于我后面安装用到的是apt-get，所以这个download link对我没有用。**license key **十分关键，先把它下载下来。



## 使用apt-get安装

#### Package Managers

参考官方的[Quick Start Guide](http://www.stardog.com/docs/#_quick_start_guide) ,使用package managers进行安装。值得吐槽的一点是，这个文档写的比较简略，一不注意就容易安装失败(这也就是我为什么要写这篇文档的原因)。	

按照Quick Start Guide，首先使用以下命令安装

```shell
curl http://packages.stardog.com/stardog.gpg.pub | apt-key add
echo "deb http://packages.stardog.com/deb/ stable main" >> /etc/apt/sources.list
apt-get update
apt-get install -y stardog
```



这里有一个坑点，就是 echo那句的执行权限可能不够，我这边就算是用sudo还是pemission denied。因此需要登录su来执行该语句。ubuntu下(其它linux系统未验证)，su的密码每次开机应该都是随机的，因此需要使用命令

```shell
sudo passwd
```

来定义一下su的密码。定义好之后，使用命令su 进入超级管理员模式，就能成功执行。

#### Package Layout

安装完毕之后，可以用systemctl启动、停止、重启stardog服务

```shell
systemctl start stardog
systemctl restart stardog
systemctl stop stardog
```

这里可能会碰到第二个坑，就是systemctl start 可能会失败。这是因为没有把license key复制进/var/opt/stardog

中，把刚才下载好的license key(bin文件)复制进去就可以了。

或许我们有些时候并不想使用systemctl来启动，那么就可以使用其自带的命令stardog-admin来启动服务

首先，将/var/opt/stardog加入到环境变量中，并设置运行参数

```shell
export STARDOG_HOME=/var/opt/stardog
export STARDOG_JAVA_ARGS="-Xmx8g -Xms8g -XX:MaxDirectMemorySize=2g"
```

然后直接输入

```shell
./stardog-admin server start
```

这步做好之后，stardog的服务就启动好了，为了在浏览器中查看，可以访问localhost:5820。然后这里又有第三个坑点，需要用户名密码。反正我是没有在文档中看到用户名密码，没办法，只好google之，默认的用户名密码都是admin(难道这是常识？）。



当浏览器跳出美丽的界面时，安装工作已经全部完成，套用文档中的一句话

> **Now go have a drink: you’ve earned it.**

