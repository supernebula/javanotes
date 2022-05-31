# Springboot整合开发实战 ：第1篇SpringBoot开发基础
# 第二章 SpringBoot基础知识

## 2.1 Spring boot启动原理

## 2.2 Spring boot 基础配置

### 2.2.2 Properties配置文件详解

默认加载classpath下的application.properties

1. PropertySource注解加载自定义配置文件
```java
@Configuration
@PropertySource("classpath:user.properties") 
//@PropertySource(value={"classpath:xxx1.properties","classpath:xxx2.properties"})
@ConfigurationProperties(prefix = "user") //读取配置文件resources/user.properties中key为user的属性
public class UserBean(){
    private String realName;

}
```
2. Value注解读取配置文件属性值
```java
@Value
```

注：application.properties 优先级最好，不要在user.properties文件中，配置相同键

### 2.2.3 YAML 配置文件详解

YAML文件名格式 xxx.yml 或 xxx.yaml
application.properties 功能通 application.yml

### 2.2.4 Spring Profiles使用说明

1. 多环境部署测试（开发、测试、生产）
Springboot多环境配置文件格式：application-{profile}.properties/yml
```file
application.properties //默认
application-dev.properties //开发
application-prod.properties //生产
```

修改环境配置文件优先级：

（1）修改application.properties如下属性
```prop
server.port=8080
spring.profiles.active=dev
```

（2）命令行参数：
```shell
java -jar xxx.jar --spring.profiles.active=prod
```

## 2.3 自定义Banner

（1）文本方式，默认文本：src/main/resource/banner.txt

（2）图片格式，默认图片：resource/banner.gif/jpg/png

application.properties
```prop
spring.banner.localtion=banner.txt
spring.banner.image.localtion=banner1.gif
```

## 2.4 内嵌式Web容器

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

### 2.4.3 Jetty配置

有点Jetty优点：采用异步Servlet，支持高并发，默认采用NIO非阻塞模型

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







