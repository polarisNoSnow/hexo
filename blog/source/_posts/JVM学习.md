---
title: JVM学习
date: 2017-07-10 10:52:00
tags:
- jdk
- java
categories:
- 技术
---


# 前言
openjdk源码构建参考：https://blog.csdn.net/hxm_Code/article/details/77417709
```
系统环境：CentOS-7-x86_64-Minimal-1708
BootStrap jdk：openjdk-1.7.0_231
目标jdk：openjdk-8u40
```
```shell
$ ./configure --with-num-cores=4 --with-target-bits=64 --with-boot-jdk=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.231-2.6.19.2.el7_7.x86_64 --with-debug-level=slowdebug --enable-debug-symbols ZIP_DEBUGINFO_FILES=0
$ make all
```
注意：具体细节或问题可以参考源码根目录下的README-builds.html

