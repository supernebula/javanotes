# 第四章Spring Boot整合持久层技术

以下需要Mysql启动包，pom.xml

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

4.1 默认连接池HiariCP

目前常用开源连接池c3p0\DBCP\Druid

SrpingBoot2.0 官方推荐数据库连接池HiariCP （开源）轻量、稳定性高、速度非常快；
SrpingBoot支持，无需单独引入，pom.xml引入JDBC即引入,；
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
```
HiariCP通过DataSourceAutoConfiguration类完成自动配置，通过DataSourceProperties类加载配置文件；

在application.properties中配置HiariCP连接池，如下

```properties
#datasource config
#驱动器类名，可以不指定，系统自动识别
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/mohai demo?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=false&serverTimezone=Asia/Shanghai&zeroDateTimeBehavior=convertToNull
spring.datasource.username=root
spring.datasource.password=123456
#spring.datasource.initialization-mode=always
#hikari数据库连接池
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
#连接池名称
spring.datasource.hikari.pool-name=Mohai_HikariCP
#最小空闲连接数，默认是10
spring.datasource.hikari.minimum-idle=5
#连接池的最大连接数，默认是10
spring.datasource.hikari.maximum-pool-size=10
#控制从连接池中获取的连接是否是自动提交事务，默认值为true
spring.datasource.hikari.auto-commit=true
#空闲连接存活的最大时间，默认为600000(10min)
spring.datasource.hikari.idle-timeout=30000
#控制池中连接的最长生命周期，值为0表示无限生命周期，默认为1800000，即30min
spring.datasource.hikari.max-lifetime=1800000
#数据库连接超时时间，默认为30s,即30000
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.connection-test-query=SELECT 1
```
注：如果mysql-connector-java用的是6.0以上，需要把com.mysql.jdbc.Driver改为com.mysql.cj.jdbc.Driver，否则会告警。com.mysql.cj.jdbc.Driver需要指定时区

配置文件后，通过改造启动类测试:
```java
@SpringBootApplication
public class CaochDemoApplication implements ApplicationRunner {
    @Autowired
    private DataSource dataSource;
    @Override
    public void run(ApplicationArguments args) throws Exception{
        try (Connection conn = dataSource.getConnection()){
            System.out.println(conn);
        }
    }
    public static void main(String[] args) {
        SpringApplication.run(CaochDemoApplication.class, args);
    }
}
```

## 4.2 配置Druid连接池

Druid包含监控、数据库连接池、SQL解析组件、SQL监控；SpringBoot中pom.xml引入：

```xml
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
        </dependency>
```
Druid通过DruidDataSourceAutoConfigure类完成自动化配置，配置参数很多，示例通过yml配置如下：

```yml
spring:
    datasource:
        # 基本属性 url、user、password， type可以不写
        # type: com.alibaba.druid.pool.DruidDataSource
        url: jdbc:mysql://127.0.0.1:3306/mohai_demo?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=true&serverTimezone=Asia/Shanghai&zeroDateTimeBehavior=convertToNull
        username: root
        password: 123456
        # 可自动跟据url识别驱动类名
        # driver-class-name: com.mysql.cj.jdbc.Driver
        # druid数据库连接池 以下参数可选都不是必须的
        druid:
            # 配置初始化大小、最小、最大
            initial-size: 1 #初始化时建立物理连接的个数，默认为0，初始化发生在显示调用init方法，或者第一次getConnection时
            min-idle: 1
            max-active: 20
            # 配置获取连接等待超时的时间 单位毫秒
            max-wait: 60000
            # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
            time-between-eviction-runsMillis: 2000
            # 配置一个连接在池中最小生存的时间，单位是毫秒
            min-evictable-idle-timeMillis: 600000
            # 配置一个连接在池中最大生存的时间，单位是毫秒
            max-evictable-idle-timeMillis: 900000
            # 检测连接是否有效的sql，如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会起作用
            validation-query: select 1
            # 检测连接是否有效的超时时间，单位：秒
            validation-query-timeout: 10
            # 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能
            test-on-borrow: false
            # 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能
            test-on-return: false
            # 建议设为true，不影响性能，申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效
            test-while-idle: true
            # 连接池中的minIdle数量以内的连接，空闲时间超过minEvictableIdleTimeMillis，则会执行keepAlive操作
            keep-alive: true
            # 是否缓存preparedStatement，也就是PSCache
            pool-prepared-statements: true
            max-open-prepared-statements: 20 #和下面一条等价
            # 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true
            max-pool-prepared-statement-per-connection-size: 20
            # 配置监控统计拦截的filters，配置多个英文逗号分隔
            filters: stat,wall
            # 通过connectProperties属性来打开mergeSql功能；慢SQL记录统计
            connection-properties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
            # 配置合并多个DruidDataSource的监控数据
            use-global-data-source-stat: true
```

## 4.3 配置MyBatis框架
MyBatis支持自定义SQL、自定义存储过程、高级映射；半自动ORM框架，SQL修改和优化灵活，低耦合；
可独立于Spring框架使用。在SpringBoot中，依赖于mybatis-spring-boot-starter模块，通过MybatisAutoConfiguration实现自动配置；

首先mybatis-spring模块，讲Mybatis无缝整合进Spring,将Mybatis事务交给Spring管理。mybatis-spring模块的SqlSessionFactoryBean类利用Spring的FactoryBean接口，通过SqlSessionFactoryBean#getObject()方法完成SqlSessionFactory的创建。

此外SqlSessionTemplate实现SqlSession接口，通过该类可以线程安全的使用SqlSession。因此常用SqlSessionTemplate来替代Mybatis默认的DefaultSqlSession。

还有个SqlSessionDaoSupport来获取SqlSessionFactory和SqlSession对象；


MybatisAutoConfiguration类加载初始化SqlSession-Template和sqlSessionFactory对象，同时初始化@Mapper注解的扫描器AutoConfiguredMapperScannerRegistrar。

用到注解，@MapperScan 、@Mapper、@Repository


