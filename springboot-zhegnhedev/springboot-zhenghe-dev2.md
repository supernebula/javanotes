# Spring Boot整合开发实战： 第二篇 第三方组件集成

## 第3章 Spring Boot整合Web开发

3.1 Spring Boot自动配置Web

3.2 配置JSON和XML数据转换

Controller
```java
@ResponseBody //根据HTTP请求头信息来判断转换器，转换为json或xml格式返回
public Object action(){}
```

3.2.1 默认转换器

默认Jackson实现JSON和XML格式数据转换。

硬编码制定转换格式：

```java

@RestController
public class UserController{
    //指定返回json
    @GetMapping(value = "/user/getJson", produces = MediaType.APPLICATION_JSON_VALUE)
    public UserVo getJson(){
        ...
    }

//指定返回xml
        @GetMapping(value = "/user/getXml", produces = MediaType.APPLICATION_XML_VALUE)
    public UserVo getXml(){
        ...
    }
}
```

注：默认返回XML会报错，需要添加xml解析工具类依赖：

```pom.xml
        <!--XML解析工具-->
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
        </dependency>
```

VO对象常用的xml注解
```java
@JacksonXmlRootElement(localName = "根节点标签名称") 
@JacksonXmlProperty(localName = "子节点标签名称"，isAttribute=true) // isAttribute=true 标签属性，isAttribute=false为子标签
@JacksonXmlElementWrapper(localName = "集合属性=外层标签名称") 
@JacksonXmlCData //标注属性，默认true, 序列化是否使用CDATA块
@JacksonXmlText //标注属性，是否生成无元素包裹的普通文本，只能用于一个属性；
```

示例：UserVo 对象序列化XML
```java
@Data
@JacksonXmlRootElement(localName = "user-info")
public class UserVo {

    @JacksonXmlProperty(localName = "real-name", isAttribute = true)
    private String realName;

    @JacksonXmlProperty
    private Integer age;

    @JacksonXmlCData
    private String remark;

    //@JacksonXmlText
    @JacksonXmlProperty
    private String title;

    @JacksonXmlElementWrapper(localName = "role-list")
    @JacksonXmlProperty(localName = "role")
    public List<RoleVo> roleList;
}
```

```java
@Data
@JacksonXmlRootElement(localName = "role-info")
public class RoleVo {

    @JacksonXmlProperty
    private String name;

    @JacksonXmlProperty
    private String code;
}
```

序列化结果：
```xml
<user-info real-name="lisi">
    <age>10</age>
    <remark>
        <![CDATA[备注信息￥%SDG]]>
    </remark>
    <title>经理</title>
    <role-list>
        <role>
            <name>管理</name>
            <code>admin</code>
        </role>
        <role>
            <name>客服</name>
            <code>service</code>
        </role>
    </role-list>
</user-info>
```



3.2.2 自定义转换器

（1）方式一，继承接口，然后Spring注入
```java
public class MyMessageConverter extends AbstractHttpMessageConverter{}
```

```java
@Configuration
public class WebMvcConfig{
    @Bean
    public MyMessageConverter myMessageConverter(){

    }
}
```
（2）方式二，实现WebMvcConfigurer

```java
@EnableWebMvc
@Configuration
public class WebMvcConfig implements WebMvcConfigurer{
}
```

（2）方式三，继承WebMvcConfigurationSupport类，重写方法

```java
@EnableWebMvc
@Configuration
public class MyWebMvcConfigSupport extentds WebMvcConfigurationSupport{
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?> converters{
        ...
    }
}
```

重写UserVo的toString()方法

注：修改pom依赖，Fastjson替换Jackson




