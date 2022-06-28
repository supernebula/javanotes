# 第7章 Spring Boot 整合Cache缓存

## 7.1 Spring  Boot 的缓存支持

Spring对缓存的支持灵活，支持SpEL（Spring Expression Language）表达式来定义缓存的key，以及书写各种条件表达式（condition）。

### 7.1.1 注解@EnableCaching开启声明式缓存

引入依赖pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

在启动类中添加@EnableCaching，启用注解缓存

```java
@EnableCaching //开启基于注解的缓存
@SpringBootApplication
public class CoachDemoFiveApplication {
    public static void main(String[] args) {
        SpringApplication.run(CoachDemoFiveApplication.class, args);
    }
}
```


@EnableCaching有两种模式，默认指定的模式是AdviceMode.PROXY
```java
AdviceMode.PROXY //
AdviceMode.ASPECTJ
```

在Controller使用缓存，不支持过期时间
```java
@RestController
public class UserController {

    @Cacheable(cacheNames = "all_user", key = ("'UserController.findAll2'"))
    @GetMapping("/findAll2")
    public List<UserDTO> findAll2(){
        return userService.findAll();
    }

    /**
    * #id 即Action的id参数
    **/
    @GetMapping("/getById/{id}")
    @Cacheable(cacheNames = "all_user", key = ("'UserController.getById'+ #id"))
    public UserDTO getById(@PathVariable Integer id){
        return userService.getOneById(id);
    }
}
```

### 7.1.2 默认的ConcurrentMapCacheManager缓存管理器

Spring Boot 默认加载使用的SimpleCacheConfiguration配置类， 且注入ConcurrentMapCacheManager。还用到了CacheAutoConfiguration。

## 7.2 EhCache缓存技术

EhCache基于内存和磁盘文件的存储方式，也可以支持分布式。 Hibernate使用EhCache实现二级缓存。纯JAVA进程内的缓存技术，提供LRU、LFU和FIFO策略。

### 7.2.1 EhCacheCacheManager缓存配置

步骤1：引入依赖pom.xml
```xml
<!--引入 Cache 缓存依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<!--引入 Ehcache 依赖-->
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<!--引入嵌入式数据库h2-->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

步骤2： 新增配置文件 resources/ehcache.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://www.ehcache.org/ehcache.xsd">
    <!--
        diskStore:表示缓存路径，ehcache分为内存和磁盘，此属性定义磁盘的缓存位置
        user.home - 用户主目录
        user.dir - 用户当前工作目录
        java.io.tmpdir - 默认临时文件路径  在windows系统下 目录为 C:\Users\登录用户\AppData\Local\Temp\ehcache
    -->
    <!-- 指定磁盘缓存位置 -->
    <diskStore path="java.io.tmpdir/ehcache" />

    <!-- 默认缓存方式
      maxElementsInMemory：缓存最大个数。
      eternal：缓存对象是否永久有效，若设置为true，timeout将被忽略，element将永不失效。
      timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒），也就是在对象消亡之前，两次访问时间的最大时间间隔。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0也就是可闲置时间无穷大。
      timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）， 也就是对象从构建到消亡的最大时间间隔。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0也就是对象存活时间无穷大。
      overflowToDisk：当内存不足时，是否启用磁盘缓存。若为true，则当内存中对象数量达到 maxElementsInMemory 时，Ehcache将会把对象写到磁盘中。
      ####以下可选属性####
      maxElementsOnDisk：硬盘缓存中可以存放的最大缓存个数，默认为0，表示无穷大。
      memoryStoreEvictionPolicy：内存存储与释放策略，当达到 maxElementsInMemory 限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。可以设置为FIFO（先进先出）或是LFU（较少使用）。
      diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
      clearOnFlush：内存数量最大时是否清除。
      ####不常用属性####
      overflowToOffHeap： 是否开启堆外缓存，只能用于企业版本中。
      maxEntriesInCache： 指定缓存中允许存放元素的最大数量，只能用在Terracotta distributed caches.（分布式缓存）
      maxBytesLocalDisk： 指定当前缓存能够使用的硬盘的最大字节数，其值可以是数字加单位，单位可以是K、M或者G，不区分大小写。
      maxBytesLocalHeap： 指定当前缓存能够使用的堆内存的最大字节数，如果设置了这个属性，maxEntriesLocalHeap 将不能被使用。
      copyOnRead：是否在读数据时取到的是Cache中对应元素的一个copy副本，而不是引用，默认false。
      copyOnWrite：是否在写入数据时用的是原对象的一个copy副本，而不是引用，默认false。
     -->
    <!--
      ####子标签元素####
      persistence：表示Cache的持久化，它只有一个属性strategy，表示当前Cache对应的持久化策略。可选值有localTempSwap、localRestartable、none、distributed。
     -->
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxElementsOnDisk="10000000"
            overflowToDisk="false"
            diskSpoolBufferSizeMB="30"
            memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap" />
    </defaultCache>

    <!--
     name:缓存名称，对应@CachePut、@CacheEvict、@Cacheable中的cacheNames/value值。
     diskPersistent：是否在VM重启时存储硬盘的缓存数据。默认值是false（ Whether the disk store persists between restarts of the Virtual Machine. The default value is false.）
     maxEntriesLocalHeap：本地堆内存中缓存的最大对象数，默认为0，表示不限制。
     maxEntriesLocalDisk：本地磁盘中保存的最大对象数，默认为0，表示不限制。
     diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认值为120秒。
    -->
    <!-- 测试用户缓存 -->
    <cache name="testUserCache"
           maxEntriesLocalHeap="10000"
           eternal="false"
           timeToIdleSeconds="3"
           timeToLiveSeconds="3"
           diskPersistent="true"
           maxEntriesLocalDisk="10000000"
           diskExpiryThreadIntervalSeconds="120"
           memoryStoreEvictionPolicy="LRU">
    </cache>
</ehcache>488888888888
```

注：有EhCacheCacheConfiguration配置类完成最终加载

步骤3：修改application.properties
```properties
#缓存类型
spring.cache.type=ehcache
#指定EhCache配置文件的路径
spring.cache.ehcache.config=classpath:ehcache.xml
```

注：虽然有以上参数说明可以参考，但是在具体使用中还要注意。对于 maxElementsnMemory属性，还有两种情况需要说明：当配置 overflowToDisk 为true时，则会将Cache中多出的元素保存到磁盘文件中；当配置 overflowToDisk 为false时，则会采用memoryStoreEvictionPolicy 配置策略替换Cache中的原有元素。另外，文件中配置的defaultCache元素用于指定Cache的默认配置，cache元素是自定义的缓存方式，但需设置name值，diskPersistent 属性和persistence元素不能同时使用。我们使用的缓存对象实体类也需要实现序列化接口，以支持磁盘存储功能。

### 7.2.2 EhCache的集群模式

EhCache使用3个方面：

1. EhCache本地缓存，直接在JVM虚拟机中进行缓存，速度快，效率高。适用于单应用或对访问要求高。 不适用于多线程，同时读写造成数据读写错误、大量对象写入磁盘阻塞。

2. 无法解决多台服务器同步的问题。

作为了解，EhCache从1.7开始支持5中集群方案：Terracotta、RMI、JMS、JGroups、EhCache Server。

## 7.3 Redis缓存技术

在pom.xml文件中引入依赖（本例引入案例JPA、H2）.

```xml
<!--引入 Cache 缓存依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<!-- 依赖Spring Boot Data Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

修改application.properties配置文件，添加属性配置。
```properties
#缓存类型
spring.cache.type=redis
#Redis数据索引（默认为0）
spring.redis.database=0
#Redis服务器地址
spring.redis.host=localhost
#Redis服务器连接端口
spring.redis.port=6379
#Redis服务器连接密码（默认为空）
spring.redis.password=
#缓存过期时间，单位为ms
spring..cache.redis.time-to-live=60000
```

注：Redis配置类RedisCacheConfiguration

基于Redis缓存的key值中的双冒号(::)，这是CacheKeyPrefix接口提供的创建自定义key值的默认实现方法。以下实现Redis配置。

### 7.3.2 Redis缓存管理

Redis缓存数据有两种持久化机制，RDB（Redis DataBase）机制、AOF（Append Only File）机制。

RDB（Redis DataBase）机制，默认的持久化机制，指定时间间隔内存写入磁盘，默认文件名dump.rdb。
AOF（Append Only File）机制将每条写入的命令都生成日志，一append-only（追加）m模式吸入一个日志文件。

仅用于缓存时，可禁用RDB、AOF。

Redis常见问题：缓存雪崩（大批数据同时过期，造成高并发穿透到数据库）、缓存穿透（查询不存在的数据，每次都查数据库）、缓存击穿（某一热点key突然过期，请求直接发送到数据库）。

解决以上问题方法是加锁、引入空值、随机缓存过期时间等。




