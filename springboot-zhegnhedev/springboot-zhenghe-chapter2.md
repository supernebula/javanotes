# Springboot整合开发实战 ：第1篇SpringBoot开发基础
# 第二章 SpringBoot基础知识

## 1.2

### 1.2.4 项目启动方式

1. 在idea中运行main() ，会自动加载classpath下的配置文件

2. 通过命令行 java -jar 启动

命令格式：
```shell
java -jar jar_path --param
例如：
java -jar coachdemo.jar --server.port=8081
```

3. 通过spring-boot-plugin启动

pom.xml文件添加如下
```xml
<project>
...
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
...
</project>
```
以上准备后，可以再idea的Terminal终端执行如下命令：
```shell
mvn spring-boot:run
```
Ctrl + C 终止应用

启动时添加参数:
```shell
mvn spring-boot:run -Dspring-boot.run.jvm-Arguments="-Dserver.port=8082"
```

启动时添加JVM参数：
```shell
mvn spring-boot:run -Dspring-boot.run.jvm-Arguments="-Dserver.port=8082 -Xms128m Xmx128m"
```

## 1.3 项目打包部署

1.3.1 打包部署
前置条件：确定pom.xml文件中添加了spring-boot-maven-plugin插件。有两种方式。
（1）在idea的Maven Projects栏中，找到module的Lifecycle，双击package，等待创建成功。
（2）在idea控制台中使用Maven命令打包，打包命令如下：
```shell
mvn clean package
```
打包时跳过测试类
```shell
mvn clean package -Dmaven.test.skip=true

```

打包后的jar包在moudle的target目录下。

在linux上当前会话启动jar 或 后台启动
```shell
java -jar example.jar
nohup java -jar example.jar &
```

1.3.2 基于Docker的简单部署

编写Dockerfile 文件
```prop
# 基础镜像使用JAVA, 从docker17.05后FROM 命令可多次使用
FROM java:8
# 指定维护者信息
MAINTAINER supernebula@gmail.com
# 指定临时目录为/tmp
VOLUME /tmp
# 将jar包添加到docker容器中并更名为app.jar
ADD example.jar app.jar
# 运行jar包
RUN bash -c 'touch /app.jar'
# 声明服务运行在8080端口
EXPOSE 8080
# 指定docker容器启动时运行jar包
ENTRYPOINT ["java", "-jar","/app.jar"， "-Djava-security.egd=file:/dev/./urandom","--spring.profiles.active=test","--server.port=8080","> /log/app.log"]

```
上传到服务,目录结构如下：
--docker
|---app.jar
|---Dockerfile

执行docker构建命令

```shell
docker build -t springboot-docker .
```

运行docker镜像为容器

```shell
docker run --name=myapp -d -p 8060:8080 springboot-docker
```

进入运行状态的容器
```shell
docker exec -it myapp bash
```

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







