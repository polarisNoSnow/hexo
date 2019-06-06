---
title: Apollo配置中心
date: 2019-06-03 16:36:58
tags:
- apollo
- java
- apns
- 配置中心
categories:
- 技术
---



## 简介
开发之初公司采用maven分环境打包，后来项目采用Spring Boot，只需新增不同环境的配置文件，如application-${env}.properties，采用命令启动时激活。不管是哪种方式，都是一次性运行完成，如果我想在程序运行之后修改默写配置参数，比如hessian地址、风控金额、规则等等，相对比较麻烦需要自己开发相关组件，我们选取了网上比较好的开源框架。

目前配置中心：Spring Cloud采用Spring Cloud Config，Dubbo +Spring Boot 采用携程的Apollo。

Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。
开源地址：[https://github.com/ctripcorp/apollo](https://github.com/ctripcorp/apollo)

如下记录了如何将配置中心整合到已有的项目中，最大程度的降低项目二次改造，配置中心服务是自己临时构建的demo，公司的配置中心由专人构建维护，此处仅用于测试。

## 环境
- Java：1.7+
- 基础框架：dubbo-spring-boot-starter
- Apollo配置中心：采用官网的[Quick Start项目](https://github.com/ctripcorp/apollo/wiki/Quick-Start)搭建（官网构建步骤明了详细，此处不再累赘）

访问Apollo配置中心[http://localhost:8070](http://localhost:8070/)，查看是否成功启动

## 项目整合

### 添加依赖包
pom文件中引入apollo-client包，用于与config server通信
```xml
<dependency>
	<groupId>com.ctrip.framework.apollo</groupId>
	<artifactId>apollo-client</artifactId>
	<version>1.1.0</version>
</dependency>
```

### 配置文件修改
对于Spring Boot的项目，只需要简单的修改配置文件
1. 在application.properties中添加app.id，相当于应用编号，配置中心以此为坐标
> app.id=your app id 

2. 在application-${env}.properties中添加Meta Server地址
> apollo.meta=http://ip:port #默认是127.0.0.1:8080

更多客户端整合方式参考：[https://github.com/ctripcorp/apollo/wiki](https://github.com/ctripcorp/apollo/wiki)

## 总结
>详情参考：https://github.com/ctripcorp/apollo/wiki/Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E8%AE%BE%E8%AE%A1

1. Apollo的基础模型
* 用户在配置中心对配置进行修改并发布
* 配置中心通知Apollo客户端有配置更新
* Apollo客户端从配置中心拉取最新的配置、更新本地配置并通知到应用

---

2. Apollo的总体设计
* Meta Server：Meta Server用于封装Eureka的服务发现接口（ConfigService和AdminService都是多实例服务，需要将它们注册到Eureka中），当于是一个Eureka Client。
* Config Service：提供配置的读取，推送功能，Apollo客户端（开发的应用程序）从这儿读取配置。
* Admin Service：提供配置的修改、发布功能，Apollo Portal（Apollo管理系统）使用该服务。
* Client和Portal通过域名访问MetaServer获取ConfigService和AdminService的服务列表（IP+Port），然后直接通过套接字访问服务。

---

3.发布流程
* 用户在Portal操作配置发布
* Portal调用Admin Service的接口操作发布
* Admin Service发布配置后，发送ReleaseMessage给各个Config Service（异步）
* Config Service收到ReleaseMessage后，通知对应的客户端