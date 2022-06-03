# Springboot整合开发实战 ：第1篇SpringBoot开发基础
# 第二章 SpringBoot基础知识

## 2.1 Spring boot启动原理

## 2.2 Spring boot 基础配置

### 2.2.2 Properties配置文件详解

默认加载classpath下的application.properties

自定义user.properties
```prop
user.realName= zhangsan3
user.age= 123
```

1. PropertySource注解加载自定义配置文件
```java
@Configuration
@PropertySource("classpath:user.properties")
//@PropertySource(value={"classpath:xxx1.properties","classpath:xxx2.properties"})
@ConfigurationProperties(prefix = "user", ignoreInvalidFields = true) //读取配置文件resources/user.properties中key为user的属性
@Data
public class UserCustomConfig {

    private String realName;

    private Integer age;
}
```

```java
@Configuration
@PropertySource("classpath:user.properties")
@Data
public class UserCustomConfig {

    @Value("${user.realName}")
    private String realName;

    @Value("${user.age}")
    private Integer age;
}
```

2. Value注解读取配置文件属性值
```java
@Value
```

注：application.properties 优先级最高，不要在user.properties文件中，配置相同键

### 2.2.3 YAML 配置文件详解

YAML文件名格式 xxx.yml 或 xxx.yaml
application.properties、 application.yml功能相同

### 2.2.4 Spring Profiles使用说明

1. 多环境部署测试（开发、测试、生产）
SpringBoot多环境配置文件格式：application-{profile}.properties(或yml)
```file
application.properties //默认
application-dev.properties //开发
application-test.properties //开发
application-prod.properties //生产
```

修改环境配置文件优先级两种方式：

（1）固定方式，修改application.properties如下属性
```prop
server.port=8080
spring.profiles.active=dev
```

（2）灵活方式，命令行参数：
```shell
java -jar xxx.jar --spring.profiles.active=prod
```

## 2.3 自定义Banner

注，Banner加载顺序：

首先依次在 Classpath下找文件banner.gif，banner.jpg和 banner.png，使用优先找到的
若没找到上面文件的话，继续 Classpath下找 banner.txt
若上面都没有找到的话, 用默认的 SpringBootBanner，也就是上面输出的 Spring Boot logo


（1）文本方式，默认文本：src/main/resource/banner.txt

（2）图片格式，默认图片：resource/banner.gif（或jpg/png）

application.properties配置自定义Banner
```prop
spring.banner.location=banner.txt
spring.banner.image.location=classpath:banner.gif
```

## 2.4 内嵌式Web容器

[Tomcat vs Jetty vs Undertow性能对比](https://cloud.tencent.com/developer/article/1699803)

### 2.4.1 SpringBoot默认Tomcat，适合中小、并发小场合；

### 2.4.2 Undertow


SpringBoot更换Undertow Web容器

```pom.xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-undertow</artifactId>
        </dependency>
```

替换后Springboot启动日志变化
```shell
2022-06-01 14:46:14.789  INFO 27308 --- [           main] io.undertow                              : starting server: Undertow - 2.0.31.Final
2022-06-01 14:46:14.800  INFO 27308 --- [           main] org.xnio                                 : XNIO version 3.3.8.Final
2022-06-01 14:46:14.818  INFO 27308 --- [           main] org.xnio.nio                             : XNIO NIO Implementation Version 3.3.8.Final
2022-06-01 14:46:14.906  INFO 27308 --- [           main] o.s.b.w.e.u.UndertowServletWebServer     : Undertow started on port(s) 8085 (http) with context path ''
```

注：Undertow使用案例少，需要自己踩坑，大部分情况Tomcat满足要求；

### 2.4.3 Jetty配置

优点Jetty优点：采用异步Servlet，支持高并发，默认采用NIO非阻塞模型

SpringBoot更换Jetty Web容器

```pom.xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
```

## 2.5 视图层技术

Thymeleaf :Springboot默认
Freemarker: 不绑定Servlet或其他Web组件
Velocity: 功能远超Web领域，可模板生成SQL、PostScript、XML







