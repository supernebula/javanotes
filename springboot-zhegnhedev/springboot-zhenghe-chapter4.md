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
运行打印日志数据源名称

```console
2022-06-11 17:14:40.295  INFO 3037 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2022-06-11 17:14:40.299  INFO 3037 --- [           main] com.evol.CoachDemoSecondApplication      : Started CoachDemoSecondApplication in 4.744 seconds (JVM running for 6.028)
2022-06-11 17:14:40.301  INFO 3037 --- [           main] com.zaxxer.hikari.HikariDataSource       : Mohai_HikariCP - Starting...
2022-06-11 17:14:40.553  INFO 3037 --- [           main] com.zaxxer.hikari.HikariDataSource       : Mohai_HikariCP - Start completed.
打印数据源名称
HikariProxyConnection@729710660 wrapping com.mysql.cj.jdbc.ConnectionImpl@550de6b8
```

## 4.2 配置Druid连接池

Druid包含监控、数据库连接池、SQL解析组件、SQL监控；SpringBoot中pom.xml引入：

```xml
        <dependency>-->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <!--SrpingBoot2.0 官方推荐数据库连接池druid 开始-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.24</version>
        </dependency>
        <!--SrpingBoot2.0 官方推荐数据库连接池druid 结束-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
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

 Spring Data JDBC借鉴DDD，通过一个聚合根（Aggregate Root）来执行持久化操作，鼓励使用领域建模；另外Spring Data项目都依赖于spring-data-commons公共模块；

 ### 4.4.1 基础配置
 引入依赖，在pom.xml包中引入spring-boot-starter-dta-jdbc包（包含spring-data-jdbc， 注解@EnableJdbcRepositories包含其中）。使用简单，实现一个接口实现CURD功能。

 ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
 ```

 通过JdbcRepositoriesAutoConfiguration实现自动化配置。通过调用CrudRepository接口提供的save()来保存聚合，实现插入和修改操作；tggg6666666666666

 ### 4.4.2 应用案例

修改pom.xml，引入依赖：
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>

        <!--配置使用Spring Data JDBC-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
```

修改application.yml配置文件，新增数据库连接配置。
```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/praxis_example?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=true&serverTimezone=Asia/Shanghai&zeroDateTimeBehavior=convertToNull
    username: root
    password: 123456
```


分别新建domain、repository、service、controller,新建如下类：

实例数据库脚本，user表：
```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '编号',
  `name` varchar(50) NOT NULL COMMENT '名称',
  `age` int(11) NOT NULL DEFAULT '0' COMMENT '年龄',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4;
```


com.evol.domain.UserEntity
```java
@Table("user")
@Data
public class UserEntity {
    @Id
    private Integer id;
    private String name;
    private int age;
}
```

com.evol.config.DataJdbcConfig
```java
@Configuration
//定义扫描的报名
@EnableJdbcRepositories("com.evol.repository")
//开启事务
@EnableTransactionManagement
public class DataJdbcConfig {
}
```

com.evol.controller.UserController
```java
@RestController
@RequestMapping("/user")
public class UserController{

    @Autowired
    private UserService userService;
    @RequestMapping("/findAll")
    public List<UserEntity>findAll(){
        return userService.getAll();
    }
    @RequestMapping("/findAllByName")
    public List<UserEntity> findAllByName(String name){
            return userService.getAllByName(name);
    }
    @RequestMapping("/save")
    public int save(@RequestBody UserEntity userEntity) {
        return userService.insertUser(userEntity);
    }
    @RequestMapping("/edit")
    public int edit(@RequestBody UserEntity userEntity)
    {
        return userService.updateUser(userEntity);
    }

    @RequestMapping("/delete")
    public int delete(@RequestParam int id){
        userService.deleteUserById(id);
        return 1;
    }
}
```

com.evol.service.UserService
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    //查
    public List<UserEntity> getAll(){
        List<UserEntity> userList = userRepository.findAll();
        return userList;
    }
    //查
    public List<UserEntity> getAllByName(String name){
        return userRepository.findByName(name);
    }

    //增
    @Transactional
    public int insertUser(UserEntity userEntity){
        return userRepository.insertNameAndAge(userEntity.getId(), userEntity.getName(), userEntity.getAge());
    }
    //改
    @Transactional
    public int updateUser(UserEntity userEntity){
        return userRepository.updateNameAndAge(userEntity.getId(), userEntity.getName(), userEntity.getAge());
    }
    //删
    @Transactional
    public void deleteUserById(Integer id){
        userRepository.deleteById(id);
    }
}
```

com.evol.repository.UserRepository
```java
public interface UserRepository extends CrudRepository<UserEntity, Integer> {
    List<UserEntity> findAll();

    @Modifying
    @Query("insert into user(id, name, age) values(:id, :name, :age)")
    int insertNameAndAge(@Param("id") Integer id, @Param("name") String name, @Param("age") int age);

    @Modifying
    @Query("update user set name = :name, age = :age where id = :id")
    int updateNameAndAge(@Param("id") Integer id, @Param("name") String name, @Param("age") int age);

    @Query("SELECT * FROM user u WHERE u.name = :name")
    List<UserEntity> findByName(@Param("name") String name);
}
```

## 4.5 配置使用Spring Data JPA

JPA 全称Java Persistence API，可以不用谢SQL实现增删改查， 是基于Hibernate实现。Hibernate 现在开发不常用，暂不展开。

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
```

用到的配置类JpaProperties。

## 4.6事务管理配置

事务 ，处理一笔交易执行的多个持久化操作。事务的4个基本特性， 原子性Atomicity、一致性Consistency、隔离性Isolation、持久性Durability简称ACID.
如果不考虑事务的隔离性，一般会出现3类现象，大致分类脏读、不可重复读、幻读。
脏读：发生同一事务时，一个事务读取另一个事务修改但为提交的数据。
幻读：也称虚读，同一事务时，多次执行同一查询返回的结果不同。

Spring中，事务的实现有两种，编程式事务和声明式事务。
声明式事务，基于AOP实现（@Transactional注解），还需在配置类中加入#EnableTransactionManagement注解实现Spring自动扫描。 @Transactional可以标注在接口、接口方法、类和类方法中。
编程时事务，则会入侵业务代码。

涉及的总要类，事务管理器接口PlatformTransactionManager、事务属性接口TransactionDefinition。

注：@Transactional对父类继承过来的方法无效，无法通过代理类识别，Spring建议在具体的实现类或被public修饰的方法中使用该注解。默认情况Spring框架的事务管理只会对运行时为检查异常的情况进行标记，发生次异常事务会回滚。

可以使用@Transactional注解定义传播机制、隔离级别和超时时间、还可控制checked Exception、unchecked Exception异常是否回滚。

@Transactional注解定义如下：

```java
import org.springframework.core.annotation.AliasFor;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import java.lang.annotation.*;

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    //和transactionManager互为别名
    @AliasFor("transactionManager")
    String value() default "";
    //指定事务管理器的限定符值

    @AliasFor("value")
    String transactionManager() default "";

    //事务传播类型
    Propagation propagation() default Propagation.REQUIRED;

    //事务隔离级别
    Isolation isolation() default Isolation.DEFAULT;

    /**事务的超时时间，单位为s
    * {@link org.springframework.transaction.TransactionDefinition}
     */
    int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

    //如果事务是只读的，允许在运行时进行相应优化
    boolean readonly() default false;

    //定义零个或多个异常，必须是Throwable的子类，指示哪些异常类型导致事务回滚
    Class<? extends Throwable>[] rollbackFor() default {};

    //定义零个或多个异常名称，必须是hrowab1e的子类，指示哪些异常类型导致事务回滚
    String[] rollbackForclassName() default {};

    //定义零个或多个异常，必须是Throwab1e的子类，指示哪些异常类型不导致事务回滚
    Class<? extends Throwable>[] noRollbackFor() default {};

    //定义零个或多个异常名称，必须是hrowab1e的子类，指示哪些异常类型不导致事务回滚
    String[] noRollbackForClassName() default {};
}
```

## 4.7多数据源配置

应对分库、分表的系统业务场景，利用多数据源在项目中实现多数据源配置、实现动态切换、读写分离的操作。

Spring Boot 2.0版本之后，配置多数据源，需要对每个数据源的所有配置进行单独配置，否则不会生效。
Spring支持多数据源配置，可以通过集成AbstractRoutingDataSource抽象类重写determineCurrentLoopupKey()方法，然后从当前线程中获取设置的数据源连接池。

AbstractRoutingDataSource继承了AbstractDataSource提供的获取数据源链接的方法。

注：配置多数据源后，事务问题可能会失效。由于AbstractRoutingDataSource只支持单个数据库事务，所以每次要在切换数据源之后开启一个事务。涉及两个及以上数据源事务时，需采用分布式事务方案。

参考：[AbstractRoutingDataSource -- Spring提供的轻量级数据源切换方式](https://www.jianshu.com/p/b158476dd33c)

具体步骤：

1. 数据源动态切换
AbstractRoutingDataSource提供了程序运行时动态切换数据源的方法，在dao类或方法上标注需要访问数据源的关键字，路由到指定数据源，获取连接。

2. 数据源切换方法
维护一个static变量datasourceContext用于记录每个线程需要使用的数据源关键字。并提供切换、读取、清除数据源配置信息的方法。







