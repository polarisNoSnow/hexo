---
title: mongodb基础
date: 2017-07-10 10:52:00
tags:
- linux
- mongodb
categories:
- 技术
---

表/集合、行/文档、字段/域
新建连接：mongod.exe ip:port/database -u user -p password
基础查询：show dbs;use database;show tables;
db.student.find({name:{$type : 2}},{}).sort({name:1}).skip(20).limit(10)
查询name域为string类型，按照name正排序，跳过20条数据，返回10条


/*********管道查询*/
aggregate()函数
下例：先匹配TXN_AMT小于1.0的数据，将筛选的数据传入到下一管道处理，根据AC_DT分组求取TXN_AMT总金额
db.getCollection('HPSTJNL_CHECKED').aggregate([{$match : {"TXN_AMT":{$lte:1.0}}},{$group:{_id : "$AC_DT", totalAmt:{$sum:"$TXN_AMT"}}}])
select AC_DT as _id, sum(TXN_AMT) as totalAmt from hc where TXN_AMT <= 1.0 group by AC_DT
//必须有一个_id，然后根据它来分组

/*创建索引(1升序，-1逆)*/
db.student.ensureIndex({KEY:1})

/*副本集设置（支持一主多从）*/
ngod --port "PORT" --dbpath "YOUR_DB_DATA_PATH" --replSet "REPLICA_SET_INSTANCE_NAME"
每个服务启动的REPLICA_SET_INSTANCE_NAME必须相同否则会不匹配
mongod --port 27017 --dbpath d:\data\master --replSet rs(主机)
mongod --port 27018 --dbpath d:\data\bak1 --replSet rs（从机）
mongod --port 27019 --dbpath d:\data\bak2 --replSet rs（仲裁：只参与投票）

主机上添加从机：rs.add("host:port")
主机上添加仲裁：rs.addArb("host:port")
从机设置可读取：rs.slaveOk(true)
如果主库宕机，此时会发生内部选举，其中一台从机成为主机，待原主机重新启动之后，会变成从机并将数据从新主机同步过来



/*集群分片搭建*/ /* http://blog.csdn.net/sharetop/article/details/53610379*/
//1. 分片服务器
mongod --port 27020 --dbpath=d:\data\shard\s0 --logpath=d:\data\shard\log\s0.log --logappend --fork(linux专属) --shardsvr(version3.4)
mongod --port 27021 --dbpath=d:\data\shard\s1 --logpath=d:\data\shard\log\s0.log --logappend --fork(linux专属) --shardsvr(version3.4)
//2. 配置服务器
mongod --port 27100 --dbpath=d:\data\shard\config --logpath=d:\data\shard\log\conf.log --logappend --fork(linux专属)  --configsvr --replSet cs(3.4版本的写法)
use admin
cfg = {
    _id:'cs',
    configsvr:true,
    members:[
        {_id:0,host:'127.0.0.1:27100'}
     ]
};
rs.initiate(cfg);
//3. 路由服务器
mongos --port 40000 --configdb cs/127.0.0.1:27100 
//4. 分片配置
//登录40000的服务，添加分片
sh.addShard('127.0.0.1:27020');
sh.addShard('127.0.0.1:27021');
//添加需要分片的库,并设置片键（设置了片键的话，每次新增不允许为空）
sh.enableSharding('polaris');
sh.shardCollection('polaris.user',{'id':1,'name':1})
//新增一定数据量的情况下，数据只出现在一个片区，（默认的配置新增10万就会出现在不同的分片）


/**********************节点说明***********************************/ //选举算法采用bully算法
primary:主节点
arbiteronly:仲裁节点,不存储数据,只参与投票
Secondary-Only:不能成为主节点,只能做为从节点,并可以参与选举
Hidden:隐藏不被链接的从节点,不被程序访问,但可以参与选举的节点
Delayed：可以指定一个时间延迟从primary节点同步数据。主要用于备份数据，如果实时同步，误删除数据马上同步到从节点，恢复又恢复不了。
Non_voting:不参与选举,只负责备分数据


/**********************其它说明**************************/
创建管理用户
use aedata
db.createUser({
    user:'admin',
    pwd:'111111',
    roles:[{role:'readWrite',db:'aedata'}]
})

//格式化
db.getCollection('HPSTJNL_CHECKED').aggregate([
{
	$match : 
		{
			TXN_AMT : 
			{
				$lte : 1.0
			}
		}
},
{
	$group : 
		{
			_id : "$AC_DT", 
			totalAmt : {
							$sum:"$TXN_AMT"
						}
		}
}
])