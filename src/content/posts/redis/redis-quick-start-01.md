---
title: Redis 快速开始 - 存储数据结构
published: 2024-05-27T12:30:00
description: 'Redis 快速开始 - 存储数据结构'
image: './assets/Logo-redis.svg'
tags: ["Redis", "缓存", "数据库"]
category: 'Redis'
draft: false 
---


### 存取 String 类型的值

与字节数组类似，Redis 字符串存储字节序列，包括文本、序列化对象、计数器值和二进制数组

```
# 存 key="bike:1"  value="Process 134"
SET bike:1 "Process 134"

# 取 key="bike:1"
GET bike:1
```

### 存取 Hash 类型的值

散列相当于是字典（dicts or hash maps）。可以使用哈希来表示普通对象并存储计数器的分组

```
> HSET bike:2 model Deimos brand Ergonom type 'Enduro bikes' price 4972
(integer) 4

> HGET bike:2 model
"Deimos"

> HGET bike:2 price
"4972"

> HGETALL bike:2
1) "model"
2) "Deimos"
3) "brand"
4) "Ergonom"
5) "type"
6) "Enduro bikes"
7) "price"
8) "4972"
```
上述命令相当于存了
```JSON
{
    "bike:2": {
        "model": "Deimos",
        "brand": "Ergonom",
        "type": "Enduro bikes",
        "price": 4972
    }
}

```

### 扫描 keyspace，查找符合特定模式的 keys

```
> SCAN 0 MATCH "bike:*" COUNT 100

1) "0"
2) 1) "bike:1"
   2) "bike:2"
```

SCAN 命令的基本语法是 SCAN cursor [MATCH pattern] [COUNT count]，其中 cursor 是游标参数，用于指示当前迭代的位置；MATCH 和 COUNT 是可选参数，用于指定模式匹配和返回的元素数量。

cursor 参数的进一步解释：
> 在 Redis 中，SCAN 命令的 cursor 参数用于指示当前迭代的位置。它是一个游标参数，用于在多次 SCAN 命令调用之间保持状态，并确保迭代的连续性。
> <br>
> 具体来说，cursor 参数的作用如下：
> <br>
> 0: 表示开始一次新的迭代。当 cursor 参数为 0 时，SCAN 命令会从数据库的起始位置开始遍历。
> <br>
> 非0值: 表示继续上一次迭代的位置。当 cursor 参数为非0值时，SCAN 命令会从上一次迭代结束的地方继续遍历。
> <br>
> 通过 cursor 参数，SCAN 命令可以在多次调用之间持续迭代数据库中的键，而不必一次性返回所有匹配的键。这种分页迭代的方式可以有效地降低对 Redis 服务器的负载，并且在大型数据库中能够更高效地处理。
