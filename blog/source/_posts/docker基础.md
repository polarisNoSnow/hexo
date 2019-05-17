---
title: docker基础
date: 2017-02-09 21:14:58
tags:
- linux
- docker
categories:
- 技术
---

# 前言
- 这些笔记都是在16年底弄的（那时候公司很还没有docker，但是国内已经很火了，很多公司都在考虑docker落地，服务上云的事情，公司后来由技术部大佬亲手构建容器平台，本身没有参与到平台搭建中，但后期的代码及部署改造经历过），当时看了下基础记了点笔记，前不久整理下放在博客里面算是纪念。--2019.05.17

  

- 目前主要的部署流程：开发人员将代码提交到gitlab之后，通过Jenkins打包代码 并执行构建脚本，在Rancher管理运行的服务，最后日志由elk收集展示搜索。（jenkins + docker + rancher + elk）

  

- 感受：改造升级之后，基本上都是一键部署，缩短了大量的上线时间，减少了因为手工上线可能带来的误操作问题。同时，伸缩扩容等 可以快速的响应流量的变化。


# 环境准备
系统环境：centos7
安装

>$ yum install docker

开启服务
>$ service docker start

测试
>$ docker run hello-world (或是docker verison)


# 镜像测试
搜索本地镜像
>$ docker images

查看源镜像库有关java方面的镜像并且收藏数大于10
>$ docker search -s 10 java 

从镜像仓库拉取镜像
>$ docker pull *** 

后台运行learn/tutorial镜像并执行sh脚本(脚本含义：每秒输出一次hello world)
>$ docker run -d learn/tutorial /bin/sh -c "while true; do echo hello world; sleep 1; done"

>$ docker run -t -i runoob/ubuntu:v2 /bin/bash  

查询正在运行的镜像
>$ docker ps

从上面的查询的ID来查看此镜像输出的日志
>$ docker logs ID

停止镜像
>$ docker stop ID

# 搭建镜像

## 安装基础系统环境

下载一个linux基础镜像
>$ docker pull centos

国外的docker Hub可能比较慢，使用下面的命令修改 daemon.json文件，会添加一个registry-mirrors：注册服务器镜像(使用daocloud的，默认为docker Hub)
>$ curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://ce611378.m.daocloud.io

查看本地镜像列表
>$ docker images

## 操作基础系统

进入系统
>$ docker run -i -t centos /bin/bash

exit退出系统----------退出系统，容器会停止，所以在此之前用此账户再开一个窗口 执行下面的操作

查询到所有的启动过容器
>$ docker ps -a 

启动停止的镜像
>$ docker start ID 

进入镜像中 
>$ docker attach ID           ###exit之后镜像依旧挂掉
或是 
>$ docker exec -i -t ID  /bin/bash   ###这种方式镜像在后台存活

## 搭建项目运行环境
JDK8 + tomcat7
- 现在宿主机器里wget下资源(JDK8+tomcat7的tar包),坑爹的JDK7官网已经不支持游客下载
- 将宿主磁盘挂载到docker镜像mnt目录下，解压然后cp到自己定义的目录
- jdk: 修改/etc/profile，添加路径保存，然后source /etc/profile更新

## 打包镜像
**建议使用dockerfile构建**

提交修改
>$ docker commit -m="第一次提交" -a="polaris" 425cef90ab3f polaris/centos:v1

-m: 描述信息
-a: 作者
425cef90ab3f：ID
polaris/centos:v1 :创建的目标镜像名+tag

重新进入刚刚提交的镜像(之前所做的操作都存在)
>$ docker run -i -t polaris/centos:v1 /bin/bash

**如果你想推送你的镜像到注册中心**
首先需要在docker官网创建一个Repository，如果你本地的名字和创建的不一样，会提示未授权。
本地打一个tag

>$ docker tag local-images:v1 dockerHub-images:v1

登录docker
>$ docker login 

然后push你的本地镜像（很多操作和git类似）
>$ docker push **** 

快速构建

- 使用docker bulid来构建（会读取dockerfile文件），更快速简洁
- 先pull然后修改之后commit（方法一的底层也是通过这种方式）

## 其它
### 更换镜像库
从国外服务器pull镜像速度较慢，直接使用由DaoCloud或者阿里提供的Registry Mirror服务
http://blog.daocloud.io/how-to-master-docker-image/

### 挂载磁盘
将宿主的下载文件夹 挂载 到docker容器polaris镜像中的docker下
>$ docker run -it -v /home/download:/mnt polaris/centos:v1 /bin/bash

没试过在已经启动的docker镜像中挂载



### 查看容器相关信息
主要是查看网络配置，容器里面很多命令都没有
>$ docker inspect a7e0139b5940

### 端口映射
交互型
>$ docker run -i -t -p 5000:8080 polaris/centos:v1 /bin/bash 

后台型
>$ docker run -d -i -t -p 5000:8080 polaris/centos:v1 

### 阿里云
以我的阿里云为例
下载
>$ docker pull registry.cn-hangzhou.aliyuncs.com/polarisnosnow/polaris:v2

上传
>$ docker login --username=****  registry.cn-hangzhou.aliyuncs.com
>$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/polarisnosnow/polaris:[镜像版本号]
>$ docker push registry.cn-hangzhou.aliyuncs.com/polarisnosnow/polaris:[镜像版本号]

### 搭建注册服务器
下载服务器镜像并运行
>$ docker pull registry 
>$ docker run -p 5000:5000 -d -i -t registry 

提交并推送
>$ docker commit cid 127.0.0.1:5000/my_image:v1
>$ docker push 127.0.0.1:5000/my_image:v1

可以使用docker API查看库中结果
http://127.0.0.1:5000/v2/_catalog
http://127.0.0.1:5000/v2/my_image/tag/list

注意客户端在/etc/docker/daemon.json 中添加{ "insecure-registries":["127.0.0.1:5000"]}
安全访问（默认走的https）

### docker管理界面
- **dockerUI**

只能用于单机，单功能齐全。

构建脚本

>$ docker run -d -p 9000:9000 --privileged -v /var/run/docker.sock:/var/run/docker.sock uifd/ui-for-docker

<br/>

- **shipyard**

适合集群，各种资源分配，性能检测等

/etc/sysconfig/docker中添加对2375的监听 

> OPTIONS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock"

shipyard构建脚本（启动的容器较多）
>$ curl -s https://shipyard-project.com/deploy | bash -s Username: admin Password: shipyard

或者
>$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock shipyard/deploy start
