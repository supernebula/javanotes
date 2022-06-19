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

#### 在springboot 使用redis

引入依赖pom.xml：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

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

org.springframework.data.redis.serizlizer包中包含各种序列化实现类。 也可以重写配置类，修改RedisTemplate的序列化方式。示例如下：

```java

@Configuration
public class RedisConfig {
    //配置自定义RedisTemplate
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnFactory){
        //创建一个RedisTemplate对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        //设置连接工厂
        template.setConnectionFactory(redisConnFactory);
        //创建Jackson2JsonRedisSerializer序列化类
        Jackson2JsonRedisSerializer serializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper mapper = new ObjectMapper(); //自定义ObjectMapper
        //指定要序列化的域和可见性
        //ALL表示可以访问所有属性，包括被private和public修饰的属性
        mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        //指定序列化输出类型，整个类除final外的属性信息都需要被序列化和反序列化
        mapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance ,
                ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.WRAPPER_ARRAY);
        serializer.setObjectMapper(mapper);

        // 使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认用的是JDK序列化）
        template.setValueSerializer(serializer);

        //配置hash的key、value序列化
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);

        // 初始化操作
        template.afterPropertiesSet();
        return template;

    }

    //也可以在此配置类中注入不同类型的数据操作对象

    //对Hash类型的数据操作
    @Bean
    public HashOperations<String, String, Object> hashOperations(RedisTemplate<String, Object> redisTemplate){
        return redisTemplate.opsForHash();
    }

    //对列表类型的数据操作
    @Bean
    public ListOperations<String, Object> listOperations(RedisTemplate<String, Object> redisTemplate){
        return redisTemplate.opsForList();
    }

    //对无序集合类型的数据操作
    @Bean
    public SetOperations<String, Object> setOperations(RedisTemplate<String, Object> redisTemplate){
        return redisTemplate.opsForSet();
    }

    //对字符串类型的数据操作
    @Bean
    public ValueOperations<String, Object> valueOperations(RedisTemplate<String, Object> redisTemplate){
        return redisTemplate.opsForValue();
    }

    //对有序集合类型的数据操作
    @Bean
    public ZSetOperations<String, Object> zSetOperations(RedisTemplate<String, Object> redisTemplate){
        return redisTemplate.opsForZSet();
    }
}

```

redis的配置application.properties
```properties
############redis
##redis客户端lettuceke配置
#Redis数据库索引（默认为0）
spring.redis.database=0
#Redis服务器地址（默认为localhost)
spring.redis.host=127.0.0.1
#Redis服务器连接端口（默认为6379）
spring.redis.port=6379
#Redis服务器连接密码（默认为空）
spring.redis.password=
#连接超时时间(ms)
spring.redis.timeout=3600
#是否使用ssL,默认为false
spring.redis.ssl=false
#连接池最大阻塞等待时间（使用负值表示没有限制），默认值为-1
spring.redis.lettuce.pool.max-wait=-1
#连接池最大连接数（使用负值表示没有限制），默认值为8
spring.redis.lettuce.pool.max-active=8
#连接池最大连接数（使用负值表示没有限制），默认值为8
spring.redis.lettuce.pool.max-idle=8
#连接池中的最小空闲连接，默认值为0
spring.redis.lettuce.pool.min-idle=0
##Jedis客户单配置
##连接池最大阻塞等待时间，单位为m。(使用负值表示没有限制)
#spring.redis.jedis.pool.max-wait=3600
##连接池最大连接数（使用负值表示没有限制），默认值为8
#spring.redis.jedis.pool.max-active=8
##连接池中的最大空闲连接，默认值为8
#spring.redis.jedis.pool.max-idle=8
##连接池中的最小空闲连接，默认值为0
#spring.redis,jedis.pool,min-idle=0
```


2. 案例2：切换Jedis客户端访问Redis

修改pom.xml
```xml
 <!--依赖Spring Boot Data Redis-->
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

        <!--对象池化组件，配合JAVA客户端的JedisPool使用-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
        <!--Jedis的jar包-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.3.0</version>
        </dependency>

```

配置Jedis配置文件
```java
@Configuration
public class JedisConfig {
    @Autowired
    private JedisConnectionFactory jedisConnectionFactory;
    //创建JedisPool
    @Bean
    public JedisPool redisPool(){
        JedisPool jedisPool = new JedisPool(jedisConnectionFactory.getPoolConfig());
        return jedisPool;
    }
}
```

```java
// todo : 测试类
```

3. 案例3：RedisTemplate的高级操作

```java
// todo : 测试类
```

### 6.2.3 Redis集群

Redis有3种集群方式：

1. master-slave（主从）模式
弊端master挂掉后，不会推选出新的master，也不会响应slave的读操作。

2. sentinel(哨兵模式)

sentinel(哨兵模式)生产环境，可实现高可用。

3. cluster（集群）模式。

如果业务量增长，可以通过cluster-enable属性开启高可用。

sentinel(哨兵模式)、cluster（集群）模式的application.properties配置如下：

```properties
#配置哨兵属性
#主节点名称
spring.redis.sentinel.master    
#以逗号分隔“主机:端口” 对列表
spring.redis.sentinel.nodes
#使用redis sentinel进行身份验证时要应用的密码
spring.redis.sentinel.password

#配置集群属性
#以逗号分隔“主机:端口” 对列表
spring.redis.cluster.nodes
#允许的集群重定向次数
spring.redis.cluster.max-redirects  
```

注：哨兵对应的配置类，RedisSentinelConfiguration.
集群模式对应的配置类，RedisClusterConfiguration







