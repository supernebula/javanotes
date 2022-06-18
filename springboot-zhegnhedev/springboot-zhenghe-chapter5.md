# 第5章 Spring Boot 构建RESTful风格

5.1 RESTful介绍

用到的注解
```java
@RequestMapping(value = "/q/{id}", method = "RequestMethod.GET")

@GetMapping  //GET

@PostMapping //POST

@PutMapping //PUT

@DeleteMapping //DELETE

```



5.2 Spring Data REST实现REST服务

Spring Data REST支持Spring Data JPA 、Spring Data MongoDB、Spring Data Neo4j、Spring Data GemFire及Spring Data Cassandra等相关数据存储，可以实现自动转换成对应资源的REST服务。

注：Spring Data REST配置信息在名为RepositoryRestMvcConfiguration中。

本示例采用H2嵌入式数据库，实现步骤：

1. 引入依赖，修改pom.xml

```xml
<!--引入Spring Data REST-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
        </dependency>

        <!--引入Spring Data JPA-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!--引入嵌入式数据库H2-->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
        </dependency>
```

2. 修改application.properties，添加如下：
```properties
spring.data.rest.base-path=/api
```

3. 创建包和类dao、domain、config

配置类config/CustomRestMvcConfiguration.class

```java
@Configuration
public class CustomRestMvcConfiguration {

    @Bean
    public RepositoryRestConfigurer repositoryRestConfigurer(){
        RepositoryRestConfigurer repositoryRestConfigurer = new RepositoryRestConfigurer() {
            public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config) {
                config.setBasePath("/api");  //将基本路径更改为 /api
                //默认为DEFAULT,现修改为ALL
                config.setRepositoryDetectionStrategy(RepositoryDetectionStrategy.RepositoryDetectionStrategies.ALL);
            }
        };
        return repositoryRestConfigurer;
    }
}
```

创建实体类和DTO类
```java
@Entity
@Data
public class AddressDTO {
    @javax.persistence.Id
    @GeneratedValue
    @Id
    private Long id;

    private final String street, zipCode, city, state;

    public AddressDTO(){
        this.street = null;
        this.zipCode = null;
        this.city = null;
        this.state = null;
    }

    public AddressDTO(String street, String zipCode, String city, String state){
        this.street = street;
        this.zipCode = zipCode;
        this.city = city;
        this.state = state;
    }

    public String toString(){
        return String.format("%s, %s, %s, %s", street, zipCode, city, state);
    }
}
```


```java
@Entity
@EntityListeners(AuditingEntityListener.class)
@Data
public class UserDTO {
    @Id
    @GeneratedValue
    private Long id;
    private String firstName;
    private String lastName;
    private String sex;

    @JsonIgnore  //可以隐藏属性
    private String password;

    @CreatedDate
    LocalDateTime createdTime;

    @LastModifiedDate
    LocalDateTime modifiedDate;

    @OneToOne(cascade = CascadeType.ALL, orphanRemoval = true)
    AddressDTO addressDTO;
}
```

创建数据层和封装域对象

```java
//自定义返回字段属性, Projection实现域对象的进一步封装
@Projection(name = "virtual", types = UserDTO.class)
public interface UserDtoExcerpt {
    String getFirstName();

    String getLastName();

    String getSex();

    @Value("#{target.firstName} #{target.lastName}")
    String getFullName();

    @Value("#target.addressDTO.toString()")
    String getAddressDTO();
}
```

```java
@RepositoryRestResource(collectionResourceRel = "user", path = "user", excerptProjection = UserDtoExcerpt.class)
public interface UserRepository extends JpaRepository<UserDTO, Long> {
    @RestResource(path = "name", rel = "name")
    List<UserDTO> findByFirstName(@Param("name") String name);
}
```

初始化数据库类，用到@PostConstruct注解，启动自动执行一次。
```java

@Component
@Slf4j
public class RestDataInit {
    @Autowired
    private UserRepository userRepository;

    //初始化，向数据库中插入一条数据
    @PostConstruct
    public void init(){
        log.debug("RestDataInit---------");
        UserDTO userDTO = new UserDTO();
        userDTO.setId(1l);
        userDTO.setSex("man");
        userDTO.setFirstName("Jack");
        userDTO.setLastName("Mr");
        userDTO.setPassword("123456");
        userDTO.setCreatedTime(LocalDateTime.now());
        userDTO.setModifiedDate(LocalDateTime.now());
        userDTO.setModifiedDate(LocalDateTime.now());
        userDTO.setAddressDTO(new AddressDTO("the five street", "123456", "New York", "Y"));
        userRepository.save(userDTO);
    }
}
```

以上代码完成

浏览器访问
GET  http://localhost:8080/api/user

提交JSON 创建对象
POST http://localhost:8080/api/user

GET访问，返回实体JSON
GET http://localhost:8080/api/user/3

## 5.3 Swagger生成API文档工具

Swagger 常用注解

```java
@Api                //作用于类
@ApiImplicitParam   //作用于方法，表单单独请求参数
@ApiImplicitParams  //作用于方法，表示多个@ApiImplicitPa的属性描述
@ApiModelProperty   //作用于方法和属性，表示POJO对象的属性描述
@ApiOperation       //作用于防范，表示Controller类中的Action方法
@ApiParam           //作用于方法、参数和属性中，单个参数描述
@ApiResponse        //作用在方法中，描述单个出参对象
@ApiResponses        //作用在类和方法中，描述多个出参对象
@ApiIgnore          //作用于类、方法和方法参数，表示忽略此参数或操作的原因
@Authorization      //作用于方法中，定义授权方案
@AuthorizationScope //作用于方法中，描述OAuth2.0授权作用域
```










