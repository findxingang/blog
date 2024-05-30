---
title: 连接到 Redis
published: 2024-05-27T12:05:00
description: '连接到 Redis'
image: '/src/assets/images/logo-redis.svg'
tags: ["Redis", "缓存", "数据库"]
category: 'Redis'
draft: false 
---



### 一、使用 Jedis 连接到 Redis

**1. 添加 Maven 依赖**

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>5.1.2</version>
</dependency>
```

**2. 连接到 Redis**
```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

import java.util.HashMap;
import java.util.Map;

/**
 * @author xingang
 * @since 2024/05/27 19:59
 */
public class Main {
    public static void main(String[] args) {
        try (JedisPool pool = new JedisPool("localhost", 6379);
             Jedis jedis = pool.getResource()) {
            // Store & Retrieve a simple string
            jedis.set("foo", "bar");
            System.out.println(jedis.get("foo")); // prints bar

            // Store & Retrieve a HashMap
            Map<String, String> hash = new HashMap<>();
            hash.put("name", "John");
            hash.put("surname", "Smith");
            hash.put("company", "Redis");
            hash.put("age", "29");
            jedis.hset("user-session:123", hash);
            System.out.println(jedis.hgetAll("user-session:123"));
            // Prints: {name=John, surname=Smith, company=Redis, age=29}
        }
    }
}
```

**3. 更简单地连接到 Redis**
官网的说法：由于为每个命令添加块 try-with-resources 可能很麻烦，因此请考虑使用 JedisPooled 作为池连接的更简单方法。

实际上我发现两者都会被 IDEA 提示加 `try-with-resources`
```java
import redis.clients.jedis.JedisPooled;

//...

JedisPooled jedis = new JedisPooled("localhost", 6379);
jedis.set("foo", "bar");
System.out.println(jedis.get("foo")); // prints "bar"
```

**4. 连接到 Redis 集群**
要连接到 Redis 集群，请使用 JedisCluster

（这里我没有测试，因为没有部署 Redis 集群）
```java
public static void main(String[] args) {
        Set<HostAndPort> jedisClusterNodes = new HashSet<HostAndPort>();
        jedisClusterNodes.add(new HostAndPort("127.0.0.1", 7379));
        jedisClusterNodes.add(new HostAndPort("127.0.0.1", 7380));
        try (JedisCluster jedis = new JedisCluster(jedisClusterNodes)) {


            // Store & Retrieve a simple string
            jedis.set("foo", "bar");
            System.out.println(jedis.get("foo")); // prints bar

            // Store & Retrieve a HashMap
            Map<String, String> hash = new HashMap<>();
            hash.put("name", "John");
            hash.put("surname", "Smith");
            hash.put("company", "Redis");
            hash.put("age", "29");
            jedis.hset("user-session:123", hash);
            System.out.println(jedis.hgetAll("user-session:123"));
            // Prints: {name=John, surname=Smith, company=Redis, age=29}
        }
    }
```

**5. 使用 TLS 连接到生产环境的 Redis**

部署应用程序时，请使用 TLS 并遵循 Redis 安全准则

在将应用程序连接到启用了 TLS 的 Redis 服务器之前，需要保证证书和私钥的格式正确

如果需要将用户证书和私钥从 PEM 格式转换为 pkcs12，请使用以下命令：
```
openssl pkcs12 -export -in ./redis_user.crt -inkey ./redis_user_private.key -out redis-user-keystore.p12 -name "redis"
```

输入密码以保护您的 pkcs12 文件

使用 JDK 附带的 keytool 将服务器 （CA） 证书转换为 JKS 格式。

```
keytool -importcert -keystore truststore.jks \ 
  -storepass REPLACE_WITH_YOUR_PASSWORD \
  -file redis_ca.pem
```

使用此段代码与 Redis 数据库建立安全连接，这段代码是一个 Java 示例，演示了如何连接到 Redis 实例并执行一些基本的操作，同时使用了 SSL/TLS 安全连接
```java
import redis.clients.jedis.DefaultJedisClientConfig;
import redis.clients.jedis.HostAndPort;
import redis.clients.jedis.JedisClientConfig;
import redis.clients.jedis.JedisPooled;

import javax.net.ssl.KeyManagerFactory;
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSocketFactory;
import javax.net.ssl.TrustManagerFactory;
import java.io.FileInputStream;
import java.io.IOException;
import java.security.GeneralSecurityException;
import java.security.KeyStore;

/**
 * @author xingang
 * @since 2024/05/27 20:20
 */
public class Main4 {
    public static void main(String[] args) throws GeneralSecurityException, IOException {
        HostAndPort address = new HostAndPort("my-redis-instance.cloud.redislabs.com", 6379);

        SSLSocketFactory sslFactory = createSslSocketFactory(
                "./truststore.jks",
                "secret!", // use the password you specified for keytool command
                "./redis-user-keystore.p12",
                "secret!" // use the password you specified for openssl command
        );

        JedisClientConfig config = DefaultJedisClientConfig.builder()
                .ssl(true).sslSocketFactory(sslFactory)
                .user("default") // use your Redis user. More info https://redis.io/docs/latest/operate/oss_and_stack/management/security/acl/
                .password("secret!") // use your Redis password
                .build();

        JedisPooled jedis = new JedisPooled(address, config);
        jedis.set("foo", "bar");
        System.out.println(jedis.get("foo")); // prints bar
    }

    private static SSLSocketFactory createSslSocketFactory(
            String caCertPath, String caCertPassword, String userCertPath, String userCertPassword)
            throws IOException, GeneralSecurityException {

        KeyStore keyStore = KeyStore.getInstance("pkcs12");
        keyStore.load(new FileInputStream(userCertPath), userCertPassword.toCharArray());

        KeyStore trustStore = KeyStore.getInstance("jks");
        trustStore.load(new FileInputStream(caCertPath), caCertPassword.toCharArray());

        TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance("X509");
        trustManagerFactory.init(trustStore);

        KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance("PKIX");
        keyManagerFactory.init(keyStore, userCertPassword.toCharArray());

        SSLContext sslContext = SSLContext.getInstance("TLS");
        sslContext.init(keyManagerFactory.getKeyManagers(), trustManagerFactory.getTrustManagers(), null);

        return sslContext.getSocketFactory();
    }
}

```

**6. 生产用途**

[Production usage](https://redis.io/docs/latest/develop/connect/clients/java/jedis/)

---

### 二、使用 Lettuce 连接到 Redis

[Lettuce guide](https://redis.io/docs/latest/develop/connect/clients/java/lettuce/)


**1. 使用 Lettuce 连接到Redis**

引入 Maven 依赖
```xml
<dependency>
    <groupId>io.lettuce</groupId>
    <artifactId>lettuce-core</artifactId>
    <version>6.3.2.RELEASE</version> <!-- Check for the latest version on Maven Central -->
</dependency>
```

Java 代码
```java
import java.util.*;
import java.util.concurrent.ExecutionException;

import io.lettuce.core.*;
import io.lettuce.core.api.async.RedisAsyncCommands;
import io.lettuce.core.api.StatefulRedisConnection;

public class Async {
  public static void main(String[] args) {
    RedisClient redisClient = RedisClient.create("redis://localhost:6379");

    try (StatefulRedisConnection<String, String> connection = redisClient.connect()) {
      RedisAsyncCommands<String, String> asyncCommands = connection.async();

      // Asynchronously store & retrieve a simple string
      asyncCommands.set("foo", "bar").get();
      System.out.println(asyncCommands.get("foo").get()); // prints bar

      // Asynchronously store key-value pairs in a hash directly
      Map<String, String> hash = new HashMap<>();
      hash.put("name", "John");
      hash.put("surname", "Smith");
      hash.put("company", "Redis");
      hash.put("age", "29");
      asyncCommands.hset("user-session:123", hash).get();

      System.out.println(asyncCommands.hgetall("user-session:123").get());
      // Prints: {name=John, surname=Smith, company=Redis, age=29}
    } catch (ExecutionException | InterruptedException e) {
      throw new RuntimeException(e);
    } finally {
      redisClient.shutdown();
    }
  }
}
```

**2. Reactive Connection**
```java
import java.util.*;
import io.lettuce.core.*;
import io.lettuce.core.api.reactive.RedisReactiveCommands;
import io.lettuce.core.api.StatefulRedisConnection;

public class Main {
  public static void main(String[] args) {
    RedisClient redisClient = RedisClient.create("redis://localhost:6379");

    try (StatefulRedisConnection<String, String> connection = redisClient.connect()) {
      RedisReactiveCommands<String, String> reactiveCommands = connection.reactive();

      // Reactively store & retrieve a simple string
      reactiveCommands.set("foo", "bar").block();
      reactiveCommands.get("foo").doOnNext(System.out::println).block(); // prints bar

      // Reactively store key-value pairs in a hash directly
      Map<String, String> hash = new HashMap<>();
      hash.put("name", "John");
      hash.put("surname", "Smith");
      hash.put("company", "Redis");
      hash.put("age", "29");

      reactiveCommands.hset("user-session:124", hash).then(
              reactiveCommands.hgetall("user-session:124")
                  .collectMap(KeyValue::getKey, KeyValue::getValue).doOnNext(System.out::println))
          .block();
      // Prints: {surname=Smith, name=John, company=Redis, age=29}

    } finally {
      redisClient.shutdown();
    }
  }
}
```

**3. Redis 集群连接**

```java
import io.lettuce.core.RedisURI;
import io.lettuce.core.cluster.RedisClusterClient;
import io.lettuce.core.cluster.api.StatefulRedisClusterConnection;
import io.lettuce.core.cluster.api.async.RedisAdvancedClusterAsyncCommands;

// ...

RedisURI redisUri = RedisURI.Builder.redis("localhost").withPassword("authentication").build();

RedisClusterClient clusterClient = RedisClusterClient.create(redisUri);
StatefulRedisClusterConnection<String, String> connection = clusterClient.connect();
RedisAdvancedClusterAsyncCommands<String, String> commands = connection.async();

// ...

connection.close();
clusterClient.shutdown();
```

...

...

...


---


### 三、Spring Boot 集成 Redis

[Baeldung](https://www.baeldung.com/spring-data-redis-properties)
[Spring 中文网](https://springdoc.cn/spring-boot-data-redis/)

**Maven 依赖**

除了 spring-boot-starter-data-redis 外，还添加了 commons-pool2 依赖，是因为我们需要使用到连接池。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.7.11</version>    
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

**配置属性**

使用 Lettuce 时，我们无需配置 RedisConnectionFactory。Spring Boot 会替我们完成。

因此，我们只需在 application.properties 文件中指定几个属性即可（适用于 Spring Boot 2.x）：

```yml
spring:
  data:
    redis:
      # 连接地址
      host: "localhost"
      # 端口
      port: 6379
      # 数据库
      database: 0
      # 用户名，如果有
      # username:
      # 密码，如果有
      # password:
      # 连接超时
      connect-timeout: 5s
      # 读超时
      timeout: 5s

      # Lettuce 客户端的配置
      lettuce:
        # 连接池配置
        pool:
          # 最小空闲连接
          min-idle: 0
          # 最大空闲连接
          max-idle: 8
          # 最大活跃连接
          max-active: 8
          # 从连接池获取连接 最大超时时间，小于等于0则表示不会超时
          max-wait: -1ms
```
**注意**：如果你使用的是 spring boot 2.x，上述配置的命名空间应该是 `spring.redis` 而不是 `spring.data.redis`。

当然，我们还可以配置很多其他属性。Spring Boot 文档中提供了[完整的配置属性列表](https://springdoc.cn/spring-boot/application-properties.html#application-properties.data.spring.data.redis.client-name)。

**Java 代码**

我们将使用的 Java Redis 客户端是 Lettuce，因为 Spring Boot 默认使用它。不过，我们也可以使用 Jedis：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

然后修改配置文件，把 lettuce 配置替换为 jedis 配置即可：

```properties
spring.data.redis.jedis.pool.enabled=true
spring.data.redis.jedis.pool.max-active=8
spring.data.redis.jedis.pool.max-idle=8
spring.data.redis.jedis.pool.max-wait=-1ms
spring.data.redis.jedis.pool.min-idle=0
spring.data.redis.jedis.pool.time-between-eviction-runs=
```

无论采用哪种方式，结果都是 `RedisTemplate` 的一个实例


**使用 StringRedisTemplate**

提示：`StringRedisTemplate` 其实就是 `RedisTemplate<String, String>`

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.core.StringRedisTemplate;
import pers.xingang.demo.service.BookService;

import javax.annotation.Resource;
import java.time.Duration;


/**
 * @author xingang
 * @since 2024/05/27 21:03
 */
@SpringBootApplication
public class Main {

    @Resource
    BookService bookService;

    @Resource
    StringRedisTemplate stringRedisTemplate;

    @Bean
    CommandLineRunner commandLineRunner() {
        // return args -> {
        //     Book book = new Book();
        //     book.setId(1L);
        //     book.setBookName("Java核心技术");
        //     book.setPrice(99.99);
        //     book.setPublishedDate(new Date());
        //     System.out.println("book创建完毕");
        //
        //     bookService.save(book);
        // };

        return args -> {
            // 设置
            this.stringRedisTemplate.opsForValue().set("title", "spring 中文网", Duration.ofMinutes(5));

            // 读取
            String val = this.stringRedisTemplate.opsForValue().get("title");
            System.out.println("val：" + val);
            // val：spring 中文网
        };
    }

    public static void main(String[] args) {

        SpringApplication.run(Main.class, args);

    }


}
```

**自定义 RedisTemplate 实现 Java 对象序列化与反序列化**

自定义一个 `JsonRedisTemplate`，用于把任意 Java 对象序列化为 json 数据存储到 Redis，并且也能够把 Redis 中的 json 数据反序列化为任意 Java 对象。

```java
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import org.springframework.stereotype.Component;

import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;


@Component
public class JsonRedisTemplate extends RedisTemplate<String, Object>{

    public JsonRedisTemplate(RedisConnectionFactory redisConnectionFactory) {

        // 构造函数注入 RedisConnectionFactory，设置到父类
        super.setConnectionFactory(redisConnectionFactory);
        
        // 使用 Jackson 提供的通用 Serializer
        GenericJackson2JsonRedisSerializer serializer = new GenericJackson2JsonRedisSerializer();
        serializer.configure(mapper -> {
            // 如果涉及到对 java.time 类型的序列化，反序列化那么需要注册 JavaTimeModule
            mapper.registerModule(new JavaTimeModule());
        });
        
        // String 类型的 key/value 序列化
        super.setKeySerializer(StringRedisSerializer.UTF_8);
        super.setValueSerializer(serializer);
        
        // Hash 类型的 key/value 序列化
        super.setHashKeySerializer(StringRedisSerializer.UTF_8);
        super.setHashValueSerializer(serializer);
    }
}
```

```java
@Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);

        // key 序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        redisTemplate.setKeySerializer(stringRedisSerializer);
        redisTemplate.setHashKeySerializer(stringRedisSerializer);


        // value 序列化
        // 如果涉及到对 java.time 类型的序列化，反序列化那么需要注册 JavaTimeModule
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(javaTimeModule);
        GenericJackson2JsonRedisSerializer serializer = new GenericJackson2JsonRedisSerializer(mapper);
        redisTemplate.setValueSerializer(serializer);
        redisTemplate.setHashValueSerializer(serializer);

        return redisTemplate;
    }
```

首先，继承 RedisTemplate<K,V>，泛型 K 表示 Redis Key 类型，一般都是 String，泛型 V 表示 Redis Value 类型，既然我们需要的是一个通用的 JSON Template，所以设置为 Object，Value 值可以是任意对象。

在构造函数中注入 RedisConnectionFactory 设置到父类，这是必须的。

然后创建GenericJackson2JsonRedisSerializer 实例，它是基于 Jackson 的 RedisSerializer 实现，用于任意 Java 对象和 JSON 字符串之间的序列化/反序列化。使用该实例作为普通 Value 和 Hash Value 的序列化/反序列化器。注意，因为序列化的对象可能包含了 java.time 类型的日期字段，如：LocalTime、LocalDate 以及 LocalDateTime，所以需要注册 JavaTimeModule。

创建测试类进行测试：
```java
import java.time.Duration;
import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

import cn.springdoc.demo.redis.JsonRedisTemplate;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class DemoApplicationTests {

    static final Logger logger = LoggerFactory.getLogger(DemoApplicationTests.class);


    // 注入 JsonRedisTemplate
    @Autowired
    JsonRedisTemplate jsonRedisTemplate;

    @SuppressWarnings("unchecked")
    @Test
    public void test() {
        
        // Map
        Map<String, Object> map = new HashMap<>();
        map.put("title", "spring 中文网");
        map.put("url", "https://springdoc.cn");
        map.put("createAt", LocalDateTime.now());
        
        // 设置 key/value
        this.jsonRedisTemplate.opsForValue().set("key1-string", map, Duration.ofMinutes(5));
        // 读取 key/value
        map = (Map<String, Object>) this.jsonRedisTemplate.opsForValue().get("key1-string");
        logger.info("map={}", map);
        
        // 设置 Hash Value
        this.jsonRedisTemplate.opsForHash().put("key2-hash", "app", map);
        // 读取 Hash Value
        map = (Map<String, Object>) this.jsonRedisTemplate.opsForHash().get("key2-hash", "app");

        logger.info("map={}", map);
    }
}
```

我们创建了一个 Map<String, Object> 对象，存储了 2 个 String 和一个 LocalDateTime 字段。然后使用 JsonRedisTemplate 把它存储为普通 Value 和 Hash Value。

存储成功后，再进行读取，反序列化为原来的 Map<String, Object> 对象。

运行测试，执行日志如下：

```
[           main] c.s.demo.test.DemoApplicationTests       : map={title=spring 中文网, url=https://springdoc.cn, createAt=2023-09-25T10:53:44.618386900}
[           main] c.s.demo.test.DemoApplicationTests       : map={title=spring 中文网, url=https://springdoc.cn, createAt=2023-09-25T10:53:44.618386900}
```




**自定义 RedisTemplate**


```java
@Bean
public RedisTemplate<Long, Book> redisTemplate(RedisConnectionFactory connectionFactory) {
    RedisTemplate<Long, Book> template = new RedisTemplate<>();
    template.setConnectionFactory(connectionFactory);
    // Add some specific configuration here. Key serializers, etc.
    return template;
}
```

**注意**：<u>RedisTemplate 是线程安全的。</u>

最后，让我们尝试在应用程序中使用它。如果我们想象一个 Book 类和一个 BookRepository，我们就可以使用 RedisTemplate 与作为后台的 Redis 进行交互，从而创建和检索书籍：

```java
@Autowired
private RedisTemplate<Long, Book> redisTemplate;

public void save(Book book) {
    redisTemplate.opsForValue().set(book.getId(), book);
}

public Book findById(Long id) {
    return redisTemplate.opsForValue().get(id);
}
```