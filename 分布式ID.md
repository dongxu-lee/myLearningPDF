# 分布式ID问题及解决方案学习笔记

## 1. 方案一：UUID
java.util.UUID.randomUUID().toString();
## 2. 方案二：数据库表
在数据库建一张表，设置自增ID，所有需要ID的请求都来该表获取最新ID
## 3. 方案三：雪花算法
0（1bit，不用）+（41bit时间戳）+（10bit机器id）+（12bit序列号）
## 4. 方案四：Redis的Incr
