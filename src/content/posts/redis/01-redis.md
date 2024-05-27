---
title: Redis 概述
published: 2024-05-27T12:00:00
description: 'Redis知识点总结'
image: './assets/Logo-redis.svg'
tags: ["Redis", "缓存", "数据库"]
category: 'Redis'
draft: false 
---

### Redis 的使用场景

[官网上提及的使用场景](https://redis.io/docs/latest/develop/get-started/)有：
- database
- cache
- streaming engine
- message broker 消息代理


[官网上给出的 quick start guide](https://redis.io/docs/latest/develop/get-started/) 有：
- [数据结构存储](https://redis.io/docs/latest/develop/get-started/data-store/)：数据结构存储在内存中
- [文件数据库](https://redis.io/docs/latest/develop/get-started/document-database/)
- [向量数据库](https://redis.io/docs/latest/develop/get-started/vector-database/)


---


### Redis 数据类型

官方文档：[Understand Redis data types](https://redis.io/docs/latest/develop/data-types/#core)

- String 字符串
  - 最基本的数据类型
  - 代表字节序列


- List 列表
  - 字符串列表
  - 按插入的顺序排序
  - key 是 String 类型， value 是 List 类型

- Set 集合
  - Set 是无序的字符串集合
  - 可以在 O(1) 的时间复杂度内实现增、删、测试存在性。换句话说，就是与元素的个数无关

- Hash 哈希
  - Hash 是一种键值对集合
  - Redis 的 Hash 类型类似于 Java 的 HashMap

- Sorted Set 有序集合
  - Sorted Set 是有序的字符串集合
  - 每个成员都与一个分数相关联，顺序由分数决定。如果分数相同，顺序由成员的字典顺序决定

- Stream 流
  - Stream 是一种只允许追加日志的数据结构
  - 按照事件发生的顺序记录事件，然后将它们联合起来进行处理

- Geospatial index 地理空间索引
  - Redis 地理空间索引对于查找给定地理半径或边界框内的位置非常有用。

- Bitmap 位图
  - 位图允许您对字符串执行按位操作

- Bitfield 位域
  - Redis 位字段有效地将多个计数器编码为字符串值。
  - 位字段提供原子获取、设置和增量操作，并支持不同的溢出策略

- HyperLogLog
  - Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。
  - HyperLogLog 是一种有名的基数计数概率算法 ，基于 LogLog Counting(LLC)优化改进得来，并不是 Redis 特有的，Redis 只是实现了这个算法并提供了一些开箱即用的 API


