---
layout: post
title: "CentOS 7/8 修改默认SSH端口（开启SELinux）"
description: "centos修改默认ssh端口的方法"
date: 2020-09-01
tags: [linux, centos7]
categories: [技术, Linux]
comments: true
---

## 使用root账户编辑ssh配置文件

`[root@izj6c209wscy81xkfbtpacz ~]# vi /etc/ssh/sshd_config`

## 输入“:/Port“，在VIM中搜索Port关键词

![image.png](/assets/images/202009/275206417.png)

找到如下内容：

![image.png](/assets/images/202009/1991690075.png)

## 取消注释，增加一行配置设置新端口

![image.png](/assets/images/202009/4132911816.png)

## 输入":/wq"保存所做的修改

![image.png](/assets/images/202009/1160132225.png)

## 在终端输入"systemctl restart sshd"重启ssh服务

![image.png](/assets/images/202009/2114609535.png)

如果没有任何输出，那么表示重启成功，可以使用新设置的端口链接服务器了（新端口22111）。

测试新端口可以连接，重新打开配置文件，在**Port 22**前方添加：**#**，注释掉，这样就无法使用22端口登入服务器。

## 把端口号加入到防火墙中，同时删除掉22端口的防火墙白名单

加入新端口：

`firewall-cmd --zone=public --add-port=22111/tcp --permanent`

`firewall-cmd --zone=public --remove-port=22/tcp --permanent`

重启firewalld服务：

`firewall-cmd --reload`

[scode type="yellow"]注意如果你开启了SELinux，需要额外执行如下命令，把端口加入到SELinux名单中[/scode]

### 安装semanage工具

`yum -y install policycoreutils-python`

输入`semanage`检查是否安装成功，输出如下表示安装成功

![image.png](/assets/images/202009/1404128175.png)

添加端口：

`semanage port -a -t ssh_port_t -p tcp 22111`

检查是否添加成功：

`semanage port -l | grep ssh_port`

![image.png](/assets/images/202009/2101443704.png)

[scode type="yellow"]如果你使用的是阿里云/腾讯云，请自行添加端口到防火墙安全组中[/scode]

修改，禁用端口完成。
