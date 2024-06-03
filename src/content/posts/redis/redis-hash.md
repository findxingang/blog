---
title: Redis：Hash
published: 2024-05-27T15:37:28
description: "Redis数据结构: 哈希"
image: "/src/assets/images/redis/logo-redis.svg"
tags: ["Redis", "缓存", "数据库"]
category: "Redis"
draft: false
---

### 基于 RedisTemplate 操作 Hash

```java

@SpringBootTest
@Slf4j
public class LettuceApplicationTests {
    @Resource
    private RedisTemplate redisTemplate;

    /**
     * RedisTemplate 操作 Hash 类型
     */
    @Test
    public void testRedisTemplate() {
        HashMap<Object, Object> hashMap = new HashMap<>();
        hashMap.put("name", "zhangsan");
        hashMap.put("age", 18);
        hashMap.put("birthdat", LocalDateTime.now());
        redisTemplate.opsForHash().putAll("crhms:dev:zhangsan", hashMap);
        Map<Object, Object> map = redisTemplate.opsForHash().entries("crhms:dev:zhangsan");
        map.forEach((k, v) -> {
            log.info("key: {}, value: {}", k, v);
        });
    }

}
```

控制台输出

```
2024-05-30 10:42:59.898  INFO 66076 --- [           main] c.e.lettuce.LettuceApplicationTests      : key: birthdat, value: 2024-05-30T10:42:57.422
2024-05-30 10:42:59.899  INFO 66076 --- [           main] c.e.lettuce.LettuceApplicationTests      : key: name, value: zhangsan
2024-05-30 10:42:59.899  INFO 66076 --- [           main] c.e.lettuce.LettuceApplicationTests      : key: age, value: 18
```

Redis 可视化工具可以看到

![redis-01](src/assets/images/redis/redis-01.png)
