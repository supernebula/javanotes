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
已拍照，图片识别，todo... 补充
```

## 4.3 配置MyBatis框架
MyBatis支持自定义SQL、自定义存储过程、高级映射；半自动ORM框架，SQL修改和优化灵活，低耦合；
可独立于Spring框架使用。在SpringBoot中，依赖于mybatis-spring-boot-starter模块，通过MybatisAutoConfiguration实现自动配置；



首先mybatis-spring模块，讲Mybatis无缝整合进Spring,将Mybatis事务交给Spring管理。mybatis-spring模块的SqlSessionFactoryBean类利用Spring的FactoryBean接口，通过SqlSessionFactoryBean#getObject()方法完成SqlSessionFactory的创建。

此外SqlSessionTemplate实现SqlSession接口，通过该类可以线程安全的使用SqlSession。因此常用SqlSessionTemplate来替代Mybatis默认的DefaultSqlSession。

还有个SqlSessionDaoSupport来获取SqlSessionFactory和SqlSession对象；


MybatisAutoConfiguration类加载初始化SqlSession-Template和sqlSessionFactory对象，同时初始化@Mapper注解的扫描器AutoConfiguredMapperScannerRegistrar。

用到注解，@MapperScan 、@Mapper、@Repository

Mybatis的属性配置类MybatisProperties，对应的application.yml配置文件如下：
```yml
mybatis:
    mapper-localtions: classpath: mappers/*.xml
    type-aliases-package: com.example.domain.model
    type-handlers-package: com.example.typehandler
    configuration:
        map-underscore-to-camel-case: true #开启驼峰转换
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl #控制台打印 SQL
        default-fetch-size: 100
        default-statement-timeout: 30
```

### 4.3.2 自定义插件

分页插件 mybatis-pagehelper，MyBatis允许在映射语句执行过程中对那些调用方法进行拦截。默认情况下使用插件拦截一下5个对象调用的方法, 数字为执行顺序：

```java
1. Executor(update, query, flushStatements, commit, rollback, getTransaction, close, isClosed);

2. StatementHandler(prepare, parameterize, batch, update, query);

3.1 ParameterHandler(getParametrObject, setParameters);

3.2 ResultSetHandler(handleResultSets, handleOutoutParameters);

```


实现自定义插件，实现Interceptor接口，并通过注解@Intercepts标识拦截器，使用注解@Signature：

```java
public interface Interceptor{
    //覆盖被拦截对象的原方法，通过Invocaton反射调用原对象的方法
    Object intercept(Invocation invocation) throws Throwable;
    //target 是被拦截的对象，主要是包装该对象生成一个代理对象
    default Object plugin(Object target){
        //默认实现逻辑，返回包装后的代理类
        return Plugin.wrap(target, this);
    }
    //该方法只会在初始化的时候会被调用一次，允许配置参数
    default void seteProperties(Properties properties){
        //NOP
    }
}

```

实现拦截器链
```java
public class InterceptorChain{

}
```
### 4.3.3 应用案例

SpringBoot 实现web和数据库，并实现自定义插件。

## 4.4 配置使用Spring Data JDBC
 





