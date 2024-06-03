---
title: Redis：String
published: 2024-05-27T13:00:00
description: "Redis数据结构:字符串"
image: "/src/assets/images/redis/logo-redis.svg"
tags: ["Redis", "缓存", "数据库"]
category: "Redis"
draft: false
---

### 基于 RedisTemplate 操作 String 类型

存和取

```java
    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @RequestMapping("/testString")
    public void testString() {
        String key = "key1";
        stringRedisTemplate.opsForValue().set(key, "这是一段文字 666");
        String value = stringRedisTemplate.opsForValue().get(key);
        System.out.println("value = " + value);
    }
```

### 数据结构

![图1](src/assets/images/redis/redis-string-01.png)

### Redis String 命令

[参考文档](https://redis.io/docs/latest/commands/?group=string)


**APPEND**
用法：如果 key 已经存在并且是字符串，则此命令将该值追加到字符串的末尾。如果 key 不存在，则创建它并将其设置为空字符串，因此 APPEND 在这种特殊情况下类似于 SET。

语法：`APPEND key value`
时间复杂度：`O(1)`

举例：
```
redis> EXISTS mykey
(integer) 0
redis> APPEND mykey "Hello"
(integer) 5
redis> APPEND mykey " World"
(integer) 11
redis> GET mykey
"Hello World"
redis> 

```