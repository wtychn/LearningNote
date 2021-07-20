# ElasticSearch

---
layout: post
title: "ElasticSearch 入门"
date: 2021-07-05 16:36
comments: true
tags: 
	- 中间件
	- 知识总结

---

## 1. 名词解释

### 1.1 接近实时（NRT） 

Elasticsearch是一个接近实时的搜索平台。这意味着，从索引一个文档直到这个文档能够被搜索到有一个轻微的延迟（通常是1秒）。

### 1.2 集群（cluster） 

一个集群就是由一个或多个节点组织在一起，它们共同持有你整个的数据，并一起提供索引和搜索功能。一个集群由一个唯一的名字标识，这个名字默认就是“elasticsearch”。这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群。

### 1.3 节点（node） 

一个节点是你集群中的一个服务器，作为集群的一部分，它存储你的数据，参与集群的索引和搜索功能。和集群类似，一个节点也是由一个名字来标识的，默认情况下，这个名字是一个随机的漫威漫画角色的名字，这个名字会在启动的时候赋予节点。这个名字对于管理工作来说挺重要的，因为在这个管理过程中，你会去确定网络中的哪些服务器对应于 Elasticsearch 集群中的哪些节点。

### 1.4 索引（index）、类型（type）、文档（document）

与关系性数据库的对应关系。

| ElasticSearch    | 关系型数据库       |
| ---------------- | ------------------ |
| 索引（index）    | 数据库（database） |
| 类型（type）     | 表（table）        |
| 文档（document） | 行                 |

### 1.5 分片（shards）和复制（replicas）

Elasticsearch提供了将索引划分成多份的能力，这些份就叫做分片。当你创建一个索引的时候，你可以指定你想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。
分片之所以重要，主要有两方面的原因：

- 允许你水平分割/扩展你的内容容量；
- 允许你在分片（潜在地，位于多个节点上）之上进行分布式的、并行的操作，进而提高性能/吞吐量。

Elasticsearch 允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片，或者直接叫复制。复制之所以重要，主要有两方面的原因：

- 在分片/节点失败的情况下，提供了高可用性。因为这个原因，注意到复制分片从不与原 / 主要（original / primary）分片置于同一节点上是非常重要的；
- 扩展你的搜索量 / 吞吐量，因为搜索可以在所有的复制上并行运行。

### 1.6 Mapping

Mapping 是 ES 中的一个很重要的内容，它类似于传统关系型数据中 table 的 schema，用于定义一个索引（index）的某个类型（type）的数据的结构。

在 ES 中，我们无需手动创建 type（相当于 table）和 mapping（相关与 schema）。在默认配置下，ES 可以根据插入的数据自动地创建 type 及其 mapping。

mapping 中主要包括字段名、字段数据类型和字段索引类型这 3 个方面的定义。

- 字段名：这就不用说了，与传统数据库字段名作用一样，就是给字段起个唯一的名字，好让系统和用户能识别。

- 字段数据类型：定义该字段保存的数据的类型，不符合数据类型定义的数据不能保存到 ES 中。下表列出的是 ES 中所支持的数据类型（大类是对所有类型的一种归类，小类是实际使用的类型）。

|      大类      |         包含的小类         |
| :------------: | :------------------------: |
|     String     |           string           |
|  Whole number  | byte, short, integer, long |
| Floating point |       float, double        |
|    Boolean     |          boolean           |
|      Date      |            date            |

- 字段索引类型：索引是 ES 中的核心，ES 之所以能够实现实时搜索，完全归功于 Lucene 这个优秀的 Java 开源索引。在传统数据库中，如果字段上建立索引，我们仍然能够以它作为查询条件进行查询，只不过查询速度慢点。而在 ES 中，字段如果不建立索引，则就不能以这个字段作为查询条件来搜索。也就是说，不建立索引的字段仅仅能起到数据载体的作用。string 类型的数据肯定是日常使用得最多的数据类型，下面介绍 mapping 中 string 类型字段可以配置的索引类型。

|   索引类型   |                             解释                             |
| :----------: | :----------------------------------------------------------: |
|   analyzed   | 首先分析这个字符串，然后再建立索引。换言之，以全文形式索引此字段。 |
| not_analyzed | 索引这个字段，使之可以被搜索，但是索引内容和指定值一样。不分析此字段。 |
|      no      |            不索引这个字段。这个字段不能被搜索到。            |

如果索引类型设置为 analyzed，在表示 ES 会先对这个字段进行分析（一般来说，就是自然语言中的分词），ES 内置了不少分析器（analyser），如果觉得它们对中文的支持不好，也可以使用第三方分析器。由于笔者在实际项目中仅仅将 ES 用作普通的数据查询引擎，所以并没有研究过这些分析器。如果将 ES 当做真正的搜索引擎，那么挑选正确的分析器是至关重要的。

## 2. SpringBoot 集成 ES

### 2.1 Spring Data Elasticsearch

#### 2.1.1 在pom.xml中添加相关依赖

```xml
<!--Elasticsearch相关依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch<artifactId>
</dependency>
```

### 2.1.2 SpringBoot 配置文件

`application.yml` 

**注意：**这里的版本是 SpringBoot 2.1.3.REALSE 和 ElasticSearch 6.2.2，ElasticSearch 和 SpringBoot 的 版本搭配非常苛刻，一定要注意。

```yaml
data:
  elasticsearch:
    repositories:
      enabled: true
    cluster-nodes: 127.0.0.1:9300 # es 的连接地址及端口号，高版本 SpringBoot 该配置已经过期
    cluster-name: elasticsearch # es 集群的名称，高版本 SpringBoot 该配置已经过期
```

### 2.1.3 pojo 注释

#### @Document

```java
//标示映射到Elasticsearch文档上的领域对象
public @interface Document {
  //索引库名次，mysql中数据库的概念
    String indexName();
  //文档类型，mysql中表的概念
    String type() default "";
  //默认分片数
    short shards() default 5;
  //默认副本数量
    short replicas() default 1;

}
```

#### @Id

```java
//表示是文档的id，文档可以认为是mysql中表行的概念
public @interface Id {
}
```

#### @Field

```java
// 对应了 es 中 mapping d
public @interface Field {
  //文档中字段的类型
    FieldType type() default FieldType.Auto;
  //是否建立倒排索引
    boolean index() default true;
  //是否进行存储
    boolean store() default false;
  //分词器名次
    String analyzer() default "";
}
//为文档自动指定元数据类型
public enum FieldType {
    Text,//会进行分词并建了索引的字符类型
    Integer,
    Long,
    Date,
    Float,
    Double,
    Boolean,
    Object,
    Auto,//自动判断字段类型
    Nested,//嵌套对象类型
    Ip,
    Attachment,
    Keyword//不会进行分词建立索引的类型
}
```

