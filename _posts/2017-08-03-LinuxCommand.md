---
layout: post
title: Linux常用命令
categories: shell
author: Kuang
tags: shell

---





本文为一些常见的Linux命令(日常生活中遇到的就记录一下)，持续更新。。。





### 安装deb包

`dpkg -i XXX.deb #`

### 修改环境变量

`vim ~/.bashrc `

### 设置系统默认jdk版本

```shell
update-alternatives --display java #查看java命令的所有可选命令
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk1.8.0_131/bin/java 300  

sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk1.8.0_131/bin/javac 300  
sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/jdk1.8.0_131/bin/jar 300   
sudo update-alternatives --install /usr/bin/javah javah /usr/lib/jvm/jdk1.8.0_131/bin/javah 300   
sudo update-alternatives --install /usr/bin/javap javap /usr/lib/jvm/jdk1.8.0_131/bin/javap 300 

sudo update-alternatives --config java

```

## 查看进程、杀死进程、启动进程

* 查看进程 

  ps a 现行终端机下所有程序，包括其他用户的程序

  ps -A 显示所有程序

  ps c 列出程序时，显示每个程序真正的指令名称，而不包含路径，参数或常驻服务的标示

  ps aux | grep XXX 查找特定进程

* 杀死进程

  kill -9 ID

  killall -9 NAME 



## 解压

* unzip

-x <文件列表>  解压缩文件，但不包括指定的file文件

-c 将解压缩的结果显示到屏幕上

-f 更新现有文件

-l 显示压缩文件内所包含的文件

-p 与-c类似，但不会对字符做转换

-t 检查压缩文件是否正确

-u 与-f参数类似，但是除了更新现有文件外，也会将压缩文件中的其他文件解压缩到目录中

-v 显示详细的信息

-z 仅显示压缩文件的备注文字

-a 文本文件做必要的字符转化

-b 不要对文本文件做字符转换

-C 文件名称区分大小写

-d <目录> 指定文件解压缩后的目录



## Ubuntu 没声音

在我机器上，执行一下pulseaudio 就行了，然而ctrl+z后，又没声音了，至今不知道什么鬼，暂时先将就用一下

## SSH登录远程服务器

ssh -p port user@host

## 安装软件意外终止导致的问题的解决方案
安装软件过程如果意外终止，会导致如下问题:
E: Could not get lock /var/lib/dpkg/lock - open (11: Resource temporarily unavailable)
E: Unable to lock the administration directory (/var/lib/dpkg/), is another process using it?
此时，只需要删除相关的文件即可:
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock



## Git

### 配置SSH

```  shell
#检查是否已经存在id_rsa.pub或者是否已经存在id_dsa.pub
cd ~/.ssh
ls
#若不存在，则新建一个SSH key
ssh-keygen -t rsa -C "your_email@example.com"
#接着会出现让你输入保存SSH key的文件名，以及让你输入密码，一般直接回车使用默认
#在github网站上新建一个SSH key,将.ssh/id_rsa.pub中的内容复制进去
#本地配置
git config --global user.name "your name"
git config --global user.email "your_email@youremail.com"
```

### 避免每次push都需要输入用户名密码

在使用git bash时，每次push都要输入用户名密码，很是麻烦，为了一劳永逸

只需输入命令  git config --global credential.helper store 即可

这样只要第一次输入用户名密码，后面每次都不需要输入了。

### 修改、合并、删除历史提交
```shell
git rebase -i commitID
git push -f


详见 [Git提交历史的修改删除合并等实践](http://blog.codingplayboy.com/2017/12/13/git-commit-operate/)


## 代理

### 让终端走代理

```shell
# 临时一个终端(ss)
export http_proxy="socks5://127.0.0.1:1080"
export https_proxy="socks5://127.0.0.1:1080"

#写入.bashrc ， 每次都跟随系统代理
export http_proxy="socks5://127.0.0.1:1080"
export https_proxy="socks5://127.0.0.1:1080"
source ~/.bashrc
```

## Docker
### 查看正在使用的端口
```shell
sudo docker ps
``` 
### 杀掉一个进程
```shell
sudo docker kill container
```

