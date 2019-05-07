---
title: keepalived笔记
date: 2017-12-15 14:08:30
tags:
- linux
- keepalived
- nginx
categories:
- 技术
---

a、keepalived安装(教程参考：https://www.cnblogs.com/Richardzhu/p/4202416.html )
1.依赖安装
c++相关：yum -y install gcc-c++
openssl: yum -y install openssl-devel
2./configure --prefix=/usr/local/keepalived --disable-fwmark 检查环境、设置、生成makefile文件
3.make && make install 编译安装
4.修改配置文件/etc/keepalived/keepalived.conf
5.service keepalived start

b、nginx安装
wget http://dl.Fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm 
rpm -ivh epel-release-latest-7.noarch.rpm
yum  -y install nginx
service nginx start 
**记得开放端口

*issue1
keepalived安装在两台物理服务器上，并相互监控对方是否在正常运行。
当节点A正常的时候:节点A上的keepalived会将下面的信息广播出去:
192.168.8.100 这个IP对应的MAC地址为节点A网卡的MAC地址
图中的其它电脑如客户端和NodeB会更新自己的ARP表，对应192.168.8.100的MAC地址=节点A网卡的MAC地址。
当节点A发生故障的时候，节点B上的keepalived会检测到，并且将下面的信息广播出去:
192.168.8.100 这个IP对应的MAC地址为节点B网卡的MAC地址
图中的其它电脑如客户端会更新自己的ARP表，对应192.168.8.100的MAC地址=节点B网卡的MAC地址

*issue2
在双机启动的时候，主备（master、backup）都会检测对方的priority，如果比对方大自己直接标记为主机；所以如果互相失联，而本身都健康则都会成为master。

*issue3
keepalived的ip漂移是当主机宕机或是keepalived挂掉，所以keepalived本身并不能因为nginx的状态随之切换，需要自己构建脚本
基本思路：1.定时检查nginx状态，当nginx关闭后，直接杀死keepalived进程（外挂定时任务或是keepalived执行脚本）
	  2.或是降低优先级也行，不过第一种稍微简单直接。

*issue4
通过查看VRRP通信原理发现VRRP基于报文实现的。master设置一定时间发送一个报文给backup如果backup没有收到就自己成为master。可推出导致问题的原因是因为backup没有收到报文，所以自己成为了master
so添加防火墙+延长发送间隔时间(keepalived.conf)
测试的时候可以直接关闭防火墙

