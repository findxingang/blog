---
title: Redis - Java 对象序列化为 JSON 字符串
published: 2024-05-29T11:16:49
description: ''
image: '/src/assets/images/logo-redis.svg'
tags: [Redis, 序列化]
category: 'Redis'
draft: false 
---

**Refrences**:

[redis序列化及各种序列化情况划分](https://www.jb51.net/article/280606.htm)

[Redis序列化存储及日期格式的问题处理](https://www.jb51.net/article/233220.htm)




### 1. 自定义序列化 RedisConfig.java

```java
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.JsonTypeInfo;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.jsontype.impl.LaissezFaireSubTypeValidator;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

/**
 * @author xingang
 */

@Configuration
public class RedisConfig {

    /**
     * 配置RedisTemplate，设置序列化器等参数
     *
     * @param connectionFactory Redis连接工厂
     * @return 配置好的RedisTemplate
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);

        // 创建一个json的序列化对象
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        // 设置对象映射器
        jackson2JsonRedisSerializer.setObjectMapper(getObjectMapper());

        // 设置key和hash key的序列化方式String
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());

        // 设置value和hash value的序列化方式json
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);

        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }

    /**
     * 配置好的ObjectMapper对象设置到JSON序列化器中
     *
     * @return ObjectMapper
     */
    private ObjectMapper getObjectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        // 设置对象的可见性，即指定哪些字段会被包含在序列化和反序列化的过程中
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 启用默认的类型信息，即在序列化结果中包含对象的类信息，但只包含非final类的信息
        // objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);   // 方法废弃 @Deprecated
        objectMapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        // 禁用日期作为时间戳的写入方式，将日期序列化为字符串形式
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        // 注册JavaTimeModule模块，用于支持Java 8的时间API
        objectMapper.registerModule(new JavaTimeModule());
        return objectMapper;
    }
}
```



### 2. 默认的序列化方式 JDK

> 即没有 RedisConfig.java
> <br>
> 也可以完成 Java 对象的序列化和反序列化，但是从 Redis 可视化工具查看时不够友好

```java
    /**
     * RedisTemplate 默认的序列化：JDK
     */
    @Test
    public void testRedisTemplateJDK() {
        User user = new User();
        user.setId("1");
        user.setName("lisi");
        user.setAge("22");
        user.setBirthday(LocalDateTime.now());

        redisTemplate.opsForValue().set("crhms:dev:lisi", user);
        User user1 = (User) redisTemplate.opsForValue().get("crhms:dev:lisi");
        log.info("user: {}", user1);
    }
```

**注意**：`User` 类需要实现 `Serializable` 接口，否则会报错无法序列化


运行结果
```
2024-05-30 13:18:03.440  INFO 45724 --- [           main] c.e.lettuce.LettuceApplicationTests      : user: User(id=1, name=lisi, age=22, birthday=2024-05-30T13:18:00.935)
```

Redis 可视化工具中查看：

![redis-02](src/assets/images/redis-02.png)