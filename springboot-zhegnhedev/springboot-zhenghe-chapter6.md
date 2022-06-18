# 第6章 Spring Boot整合NoSQL

## 6.2 集成Redis数据库

Redis基于主从（Master-Slave）复制为基础，通过Redis哨兵（Sentinel）和自动化分区（Cluster集群）提供高可用性（High Availability）.

Redis支持数据持久化，通过RDB和AOF方式，保持到磁盘，重启后加载到内存。

#### 常用值类型：

字符串 String，二进制安全，一个键做大容量512MB；

列表 list，按插入循序排序字符串列表，通过链表linked List实现。常见操作LPUSH\RPUSH\LRANGE。

集合 Set，不重复且无序的字符串集合，通过哈希表实现；

有序集合 Sorted Set，有序且不允许有重复成员的集合， ZSet。

哈希 Hash，用于存储键值对的集合，适合存储对象。

Redis 2.8.9版本添加了HyperLogLog结构。

### 6.2.2 应用案例

Spring Boot Data Redis依赖于Jedis或Lettuce，本质对Redis客户端的封装。

Jedis（SpringBoot 1.0默认）客户端示例不是线程安全，需通过连接池使用Jedis，不支持异步。Lettuce（SpringBoot2.0版本默认）基于Netty矿建，方法调用异步，线程安全。

Redis使用规范：

key键名设计：命名简洁，同时可读性、可管理性；

value值设计：不使用特别大的键，减少网络流量，String类型控制来10K以内，Hash、List、Set和Zset的元素个数不要超过5000个。

1. 案例1： 使用RedisTemplate安全操作Redis

RedisTemplate比Jedis多了自动管理连接池的功能。提供各种对Redis操作、异步处理以及序列化方法，支持发布订阅功能。

RedisTemplate提供的操作接口：
```java
GeoOperations            //地理空间操作
HashOperations           //哈希操作
HyperLogLogOperations    //LogLogOperations操作
ListOperations           //列表操作
SetOperations            //无序结合操作
ValueOperations          //字符串或值操作
ZSetOperations           //有序集合操作
```

SpringBoot中相关的类：

RedisAutoConfiguration.class,其中启用属性注入，属性类RedisProperties.class。会导入LettuceConnectionConfiguration （优先Lettuce）和JedisConnectionConfiguration配置类；这两个客户端都会注入RedisConnectionFactory链接工厂类， 它们都集成自RedisConnectionConfiguration抽象类。

RedisAutoConfiguration.class通过@Bean定义两个Bean，RedisTemplate和SimpleRedisTemplate模板。

RedisTemplate默认序列化实现类为JdkSerializationRedisSerializer, SimpleRedisTemplate默认的系列化实现类为StringRedisSerializer。

注： SimpleRedisTemplate写入Redis的数据，无法用RedisTemplate读取。SimpleRedisTemplate适用于字符串类型数据存取，RedisTemplate适用于复杂对象存取。

方便起见，正常情况使用泛型为<String, Object>形式的RedisTemplate。

org.springframework.data.redis.serizlizer包中包含各种序列化实现类。 也可以重写配置类，修改RedisTemplate的序列化方式。

示例，修改RedisTemplate的序列化方式

```java


```


