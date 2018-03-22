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

  ps aux $\vert$ grep XXX 查找特定进程

* 杀死进程

  kill -9 ID

  killall -9 NAME 

### 查看服务运行状态
```shell
#查看某个服务的状态
service servicename status
#查看所有服务
service --status-all
```

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



## SSH登录远程服务器
A机器登录到B机器
首先在A机器生成公钥私钥对
```shell
ssh-keygen -t rsa
```
然后在B机器建立.shh目录
```shell
mkdir /root/.ssh
```
将A机器上生成的公钥复制到B机器
```shell
scp -P port ~/.ssh/id_rsa.pub root@ip:/root/.ssh/authorized_keys
```
B机器更改authorized_keys权限
```shell
chmod 600 /root/.ssh/authorized_keys
```

最后测试能否登录
ssh -p port user@host

> 如果出现以下问题，说明之前已经登录过B机器，但是B机器发生了一些改变，导致无法登录
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:D6IzPjNVZ1vqaOWXJb+R6NEjNt/l69PBbqAtL6Yq/40.
Please contact your system administrator.
此时只需要删除A机器上的known_hosts即可：rm -rf ~/.ssh/known_hosts


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
```

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
>注:这个周末shadowsocks坏了，重装了一下vps系统，结果ubuntu上不去外网了，折腾了一天，最后发现是原来的shadowsocks-qt5出了问题，更新到最新就好了，这期间试过polipo等工具都不好用，还是上述方法好用

###shadowsocks配置
(该部分来自博客[https://blog.whsir.com/post-274.html](https://blog.whsir.com/post-274.html) )
ssh到vps后，执行如下命令
```shell
yum -y install wget
wget --no-check-certificate http://blog.whsir.com/uploads/ss.sh
chmod +x ss.sh
./ss.sh 2>&1 | tee shadowsocks.log
```
运行成功会有如下提示
#############################################################
# One click Install Shadowsocks(Python)
# Intro: http://blog.whsir.com
#
# Author: whsir
#
#############################################################

Please input password for shadowsocks:
(Default password: whsir):
安装完成后显示内容如下：

Congratulations, ss install completed!
Your Server IP:your_server_ip
Your Server Port:443
Your Password:your_password
Your Local IP:127.0.0.1
Your Local Port:1080
Your Encryption Method:aes-256-cfb

Welcome to visit:blog.whsir.com
Enjoy it!
```
卸载方法:
./ss.sh uninstall
多端口多密码配置：
```shell
#vi /etc/shadowsocks.json
```
配置
```shell
{
"server":"0.0.0.0",
"local_address":"127.0.0.1",
"local_port":1080,
"port_password":{
"7788":"password0",
"7789":"password1",
"7790":"password2"
},
"timeout":300,
"method":"aes-256-cfb",
"fast_open": false
}
```
需要用到的命令：
启动：service shadowsocks start
停止：service shadowsocks stop
重启：service shadowsocks restart
状态：service shadowsocks status


###　kcptun加速
该部分来自博客[https://blog.whsir.com/post-298.html](https://blog.whsir.com/post-298.html)
```shell
#mkdir /root/kcptun

#cd /root/kcptun

#wget https://github.com/xtaci/kcptun/releases/download/v20160922/kcptun-linux-386-20160922.tar.gz

#tar -zxvf kcptun-linux-386-20160922.tar.gz
```
创建启动服务：

#vi /root/kcptun/start-kcptun.sh
```shell
#!/bin/bash
cd /root/kcptun/
./server_linux_386 -l :2006 -t 23.83.229.171:7777 -key test -mtu 1400 -sndwnd 2048 -rcvwnd 2048 -mode fast2 > kcptun.log 2>&1 &
echo "Kcptun started"
```
监听端口为2006，这个数字是你自己起的，7777为你当然的ss端口
创建关闭服务：

```shell
#vi /root/kcptun/stop-kcptun.sh
killall server_linux_386
echo "Kcptun stoped"
```

启动服务
```shell
#sh /root/kcptun/start-kcptun.sh
```
配置开机自启动
```shell
echo "sh /root/kcptun/start-kcptun.sh" >> /etc/rc.local
```

windows客户端配置
载kcptun-windows-amd64-20160922

下载后解压到一个指定的文件夹内，在此文件夹内创建文本文件，内容如下：
```shell
Dim RunKcptun
Set fso = CreateObject("Scripting.FileSystemObject")
Set WshShell = WScript.CreateObject("WScript.Shell")
'获取文件路径
currentPath = fso.GetFile(Wscript.ScriptFullName).ParentFolder.Path & "\"
'软件运行参数
exeConfig = "client_windows_amd64.exe -l :12300 -r 23.83.229.171:2006 -key test -mtu 1400 -sndwnd 256 -rcvwnd 2048 -mode fast2 -dscp 46"
'日志文件
logFile = "kcptun.log"
'拼接命令行
cmdLine = "cmd /c " & currentPath & exeConfig & " > " & currentPath & logFile & " 2>&1"
'启动软件
WshShell.Run cmdLine, 0, False
'等待1秒
'WScript.Sleep 1000
'打印运行命令
'Wscript.echo cmdLine
Set WshShell = Nothing
Set fso = Nothing
'退出脚本
WScript.quit
```
保存退出，将此文本文件重命名为run.vbs，这个run.vbs就是客户端的启动程序；
>注意：这里的12300为你自己起的一个端口，这个12300和之前所有端口都没关系，不能和之前的端口号重复！！！2006是你之前的监听端口，key为之前设置的test！！！

创建一个关闭程序：

新建文本文件，内容为：taskkill /f /im client_windows_amd64.exe 保存退出，重命名为stop.bat

设置shadowsocks：

服务器IP为127.0.0.1

服务器端口为12300

密码为你的ss密码（和原来你用ss时候的密码一样）

双击run.vbs启动服务，服务启动后，打开任务管理器可在进程中看到此服务，即表示服务启动完成：

Android端配置:
datashard、parityshard、nocomp、key、crypt，配置的时候保证客户端和服务端一致即可。
[示例](https://raw.githubusercontent.com/CrisJk/crisjk.github.io/master/resource/pictures/Shadowsocks_Android_kcptun_2-550x978.png)


### 路由追踪
```shell
traceroute ip

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

##端口
### 查看端口占用情况
```shell
sudo lsof -i
#查看指定端口
sudo lsof -i :端口号
#查看已经连接的服务端口
netstat -a
查看所有的服务端口（LISTEN，ESTABLISHED）
netstat -ap
```



