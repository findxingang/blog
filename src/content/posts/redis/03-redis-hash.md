---
title: Redis：Hash
published: 2024-05-27T15:37:28
description: 'Redis数据结构: 哈希'
image: './assets/Logo-redis.svg'
tags: ["Redis", "缓存", "数据库"]
category: 'Redis'
draft: false 
---


### 基于 RedisTemplate 操作 Hash

存取
```java
    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    @RequestMapping("/testHash")
    public void testBook() {
        Book book = new Book();
        book.setId(10086L);
        book.setBookName("Java从入门到放弃");
        book.setPrice(88.88);
        book.setPublishedDate(new Date());

        Map<String, Object> hash = new HashMap<>();
        hash.put("id", book.getId());
        hash.put("bookName", book.getBookName());
        hash.put("price", book.getPrice());
        hash.put("publishedDate", book.getPublishedDate());

        redisTemplate.opsForHash().putAll(book.getBookName(), hash);
        Map<Object, Object> objectMap = redisTemplate.opsForHash().entries(book.getBookName());
        System.out.println("objectMap = " + objectMap);
        // 打印: objectMap = {publishedDate=Tue May 28 17:53:43 CST 2024, price=88.88, id=10086, bookName=Java从入门到放弃}
    }
```