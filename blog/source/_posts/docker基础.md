---
title: docker基础
date: 2017-02-09 21:14:58
tags:
- linux
- docker
categories:
- 技术
---

# 环境准备
系统环境：centos7
安装：

>$ yum install docker

开启服务：service docker start
测试：docker run hello-world (或是docker verison)

# 镜像测试
搜索本地镜像：docker images
查看源镜像库有关java方面的镜像并且收藏数大于10：docker search -s 10 java 
下载镜像：docker pull *** （直接运行某个镜像的时候也会自动下载）

后台运行learn/tutorial镜像并执行sh脚本(脚本含义：每秒输出一次hello world)
docker run -d learn/tutorial /bin/sh -c "while true; do echo hello world; sleep 1; done"

docker run -t -i runoob/ubuntu:v2 /bin/bash  

docker ps 查询正在运行的镜像

从上面的查询的ID来查看此镜像输出的日志
docker logs ID

停止镜像
docker stop ID

# 项目实战(注意各个版本)
docker容器环境的安装

## centos的安装
docker pull centos
国外的docker Hub可能比较慢，使用下面的命令修改 daemon.json文件，会添加一个registry-mirrors：注册服务器镜像(使用daocloud的，默认为docker Hub)
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://ce611378.m.daocloud.io

查看本地镜像列表
docker images

## 操作centos
进入系统：docker run -i -t centos /bin/bash
新建文件：mkdir test 
！！！然后更新yum update(ubuntu的系统是 apt-get update)--------教程中只是做一个修改，直接mkdir
！！！然后exit退出系统----------退出系统，容器会停止，所以在此之前用此账户再开一个窗口 执行下面的操作
(后来发现docker ps -a 可以查询到所有的启动过容器，可以获取到ID。
 docker start ID 启动停止的镜像
 docker attach ID 进入镜像中            ###exit之后镜像依旧挂掉
 或是 docker exec -i -t ID  /bin/bash   ###这种方式镜像在后台存活
)

## 提交修改
docker commit -m="第一次提交" -a="polaris" 425cef90ab3f polaris/centos:v1
-m: 描述信息
-a: 作者
425cef90ab3f：ID
polaris/centos:v1 :创建的目标镜像名+tag

重新进入刚刚提交的镜像
docker run -i -t polaris/centos:v1 /bin/bash
之前所做的操作都存在

### \*\*\*如果你想推送你的镜像到注册中心
首先需要在docker官网创建一个Repository
如果你本地的名字和创建的不一样，会提示未授权，使用docker tag local-images:v1 dockerHub-images:v1
a、docker login 登录docker
b、docker push **** 然后push你的本地镜像（很多操作和git类似）


快速构建
1、使用docker bulid来构建（会读取dockerfile文件），推荐！更快速简洁
2、先pull然后修改之后commit（方法一的底层也是通过这种方式）

####\*\* 注意 \*\*
## 1、从国外服务器pull镜像速度较慢，直接使用由DaoCloud或者阿里提供的Registry Mirror服务
http://blog.daocloud.io/how-to-master-docker-image/

## 2、挂载磁盘
docker run -it -v /home/download:/mnt polaris/centos:v1 /bin/bash
将宿主的下载文件夹 挂载 到docker容器polaris镜像中的docker下
但如果你想 在已经启动的docker镜像中挂载 请参考（还没试过）http://www.open-open.com/lib/view/open1421996521062.html

######3、搭建基本环境(JDK8 + tomcat7)
a, 现在宿主机器里wget下资源(JDK8+tomcat7的tar包),坑爹的JDK7官网已经不支持游客下载
b, 将宿主磁盘挂载到docker镜像mnt目录下，解压然后cp到自己定义的目录
c, jdk: 修改/etc/profile，添加路径保存，然后source /etc/profile更新

######4、查看容器相关信息（主要是查看网络配置，容器里面很多命令都没有）
docker inspect a7e0139b5940

######5、端口映射
docker run -i -t -p 5000:8080 polaris/centos:v1 /bin/bash （交互型）
docker run -d -i -t -p 5000:8080 polaris/centos:v1 （后台型）

######6、阿里云（以本人为例子）
下载
docker pull registry.cn-hangzhou.aliyuncs.com/polarisnosnow/polaris:v2

上传
docker login --username=****  registry.cn-hangzhou.aliyuncs.com
docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/polarisnosnow/polaris:[镜像版本号]
docker push registry.cn-hangzhou.aliyuncs.com/polarisnosnow/polaris:[镜像版本号]

######7、搭建注册服务器
docker pull registry 
docker run -p 5000:5000 -d -i -t registry 
之后就可以直接commit&push了
docker commit cid 127.0.0.1:5000/my_image:v1
docker push 127.0.0.1:5000/my_image:v1

可以使用docker API查看库中结果
http://127.0.0.1:5000/v2/_catalog
http://127.0.0.1:5000/v2/my_image/tag/list

注意客户端在/etc/docker/daemon.json 中添加{ "insecure-registries":["127.0.0.1:5000"]}
安全访问（默认走的https）

######8、docker管理界面
dockerUI:只能用于单机，单功能齐全
构建脚本：docker run -d -p 9000:9000 --privileged -v /var/run/docker.sock:/var/run/docker.sock uifd/ui-for-docker

shipyard:适合集群，各种资源分配，性能检测等
/etc/sysconfig/docker中添加对2375的监听 OPTIONS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock"
shipyard构建脚本（启动的容器较多） curl -s https://shipyard-project.com/deploy | bash -s
Username: admin Password: shipyard
或者：docker run --rm -v /var/run/docker.sock:/var/run/docker.sock shipyard/deploy start
