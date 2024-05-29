---
title: Redis：String
published: 2024-05-27T13:00:00
description: 'Redis数据结构:字符串'
image: './assets/Logo-redis.svg'
tags: ["Redis", "缓存", "数据库"]
category: 'Redis'
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
