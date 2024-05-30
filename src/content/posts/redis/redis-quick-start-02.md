---
title: Redis 快速开始 - 使用 Redis 作为文档数据库
published: 2024-05-27T12:31:00
description: Redis 快速开始 - 使用 Redis 作为文档数据库
image: '/src/assets/images/logo-redis.svg'
tags: [Redis, 缓存, 数据库]
category: 'Redis'
draft: false 
---

[原文链接](https://redis.io/docs/latest/develop/get-started/document-database/)

### 前置知识

**举一个例子，来解释说明什么是文档数据库**

假设你正在开发一个博客平台，需要存储用户发布的文章。在传统的关系型数据库中，你可能会设计一个包含标题、内容、作者等字段的表格来存储文章信息。但是，随着博客平台的发展，文章的结构可能会变得更加复杂，比如增加标签、评论、点赞等功能，这时关系型数据库可能会显得不太灵活。

在这种情况下，你可以选择使用文档数据库来存储文章信息。文档数据库以文档（Document）作为基本存储单元，每个文档可以包含丰富的数据结构，比如键值对、嵌套文档、数组等。以下是一个使用文档数据库的示例：

```json
{
  "title": "如何学习编程",
  "content": "学习编程的路上充满了挑战，但也是充满乐趣的旅程。",
  "author": {
    "name": "小明",
    "email": "xiaoming@example.com"
  },
  "tags": ["编程", "学习", "技术"],
  "comments": [
    {
      "author": "小红",
      "content": "很有启发性的文章，谢谢分享！"
    },
    {
      "author": "小李",
      "content": "我也在学习编程，感觉受益匪浅。"
    }
  ]
}

```
在这个示例中，每篇文章都是一个文档，包含了标题、内容、作者、标签、评论等信息。文档数据库提供了灵活的数据模型，可以轻松地存储和查询这种半结构化的数据，使得你可以更方便地处理不同类型的文章信息。

一些流行的文档数据库包括 MongoDB、Couchbase、CouchDB 等。

<br>

**Redis Stack是什么，与Redis的关系是什么**

Redis Stack 是一个构建在 Redis 数据库之上的扩展，它允许你声明哪些字段会自动创建索引，从而将 Redis 转变为一个文档数据库。Redis Stack 支持在哈希（Hash）和 JSON 文档上创建二级索引，使得你可以方便地通过这些索引来进行数据查询和检索。

Redis Stack 与 Redis 的关系是，它是基于 Redis 构建的一个功能扩展。Redis 本身是一个内存数据库，提供了多种数据结构和丰富的命令操作，但并不直接支持文档数据库的功能。Redis Stack 则在 Redis 的基础上实现了一些文档数据库的特性，如自动索引和查询功能，从而使得 Redis 可以更灵活地处理文档型数据。


### 创建二级索引
想执行以下命令时，Redis Insight 提示不支持，暂时跳过这部分
```
FT.CREATE idx:bicycle ON JSON PREFIX 1 bicycle: SCORE 1.0 SCHEMA $.brand AS brand TEXT WEIGHT 1.0 $.model AS model TEXT WEIGHT 1.0 $.description AS description TEXT WEIGHT 1.0 $.price AS price NUMERIC $.condition AS condition TAG SEPARATOR ,
```
