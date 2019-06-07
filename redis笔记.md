#  Redis学习笔记

[TOC]

##   redis基础数据结构

###   string(字符串)

####  键值对：

	set <key> <value>   
	get <key>
	exist <key> 是否存在key
	del <key>

####  批量键值对：

	mset <key1> <value1> <key2> <value2> <key3> <value3> ……
	mget <key1> <key2> <key3>
####  过期和set命令设置：

	expire <key> <time> 设置某个key的过期时间
	setex <key> <time> <value> 设置某个key的过去时间(set + expire)
	setnx <key> <value>  如果已经存在则返回0 可做分布式锁

####   计数：

	incr <key> 某个key自增1
	incrby <key> <number> 某个key增加number数量、



###  List（列表）

####  右进左出(队列)

	rpush <key> <value> <value> …… 从右边进
	llen <key> 长度
	lpop <key> 从左边弹出一个

####   右进右出(栈)

	rpop <key> 右边弹出

####  慢操作

	lindex <key> <index> 获取下标的value #O(n) 慎用
	lrange <key> 0 -1 获取所有元素 #O(n) 慎用
	ltrim <key> 1 -1 对区间(1，-1)保留，其他删除

####  快速列表

![](resource/picture/quicklist.png)



###    hash(字典)

	hset <key> <field> <value>	增加返回1，新增返回0
	hget <key> <field> 获取value
	hgetall <key> 	 # entries()，key 和 value 间隔出现
	hlen <key>	获取key数量
	hincrby <key> <field> <number> 增加number的数量



###   set(集合)

	sadd <key> <value> <value> …… 返回插入成功的数量
	smembers <key> 返回所有的value
	smember <key> <value> 看value是否存在
	scard <key> 查询数量
	spop <key> 随机弹出一个



###  zset(有序集合)

	zadd <key> <source> <value> 原先没有返回1，原先有返回0并更新source
	zrange <key> <start> <end> 按照source正序输出
	zrevrange <key> <start> <end> 按照source倒叙输出
	zcard <key> 查看有多少个key
	zscore <key> <value> 查看某个value的分数
	zrangebyscore <key> <start> <end> 通过score排序
	zrevrangebyscore <key> <start> <end> 
	zrem <key> <value> 删除某个value



###  过期时间

	ttl <key> 获取时间时间离现在秒数s