# Spring Boot整合开发实战： 第二篇 第三方组件集成

# 第3章 Spring Boot整合Web开发

## 3.1 Spring Boot自动配置Web

#  3.2 配置JSON和XML数据转换

Controller
```java
@ResponseBody //根据HTTP请求头信息来判断转换器，转换为json或xml格式返回
public Object action(){}
```

### 3.2.1 默认转换器

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

    //不指定格式，根据HTTP请求头，返回指定JSON或xml格式，添加如下2个请求头：
    //Content-Type=application/xml
    //Accept=application/xml
    @GetMapping("/user/getObj")
    public UserVo getObj(){
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

## 3.3 配置Servlet、Filter、Listener

注册方式1: ServletReqistrationBean、FilterRegistrationBean、ServletListenerRegistrationBean

注册方式2： @ServletComponentScan

Servlet3.0 @WebServlet、@WebFilter、@WebListener

1. 采用编码方式注册

自定义Servlet
```java
public class FirstServlet extends HttpServlet {

    //处理get请求
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        System.out.println("FirstServlet执行doGet");
        doPost(req, resp);
    }

    //处理post请求
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        System.out.println("FirstServlet执行doPost");
        PrintWriter out = resp.getWriter();
        out.println("FirstServlet outer");
    }

    @Override
    public void init(ServletConfig config) throws ServletException {
        System.out.println("FirstServlet init");
        super.init(config);
    }

    @Override
    public void destroy(){
        System.out.println("FirstServlet destroy");
        super.destroy();
    }
}
```
自定义Filter
```java
public class FirstFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("FirstFilter");
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("FirstFilter init");
    }

    @Override
    public void destroy() {
        System.out.println("FirstFilter destroy");
    }
}
```

自定义Listener
```java
public class FirstListenter implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("FirstListenter正在初始化");
        System.out.println("servlet container:" + sce.getServletContext().getServerInfo());
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("FirstListenter正在销毁");
    }
}
```

定义注册配置类
```java
@Configuration
public class WebConfig {

    //使用代码注册Servlet （不使用@ServletComponentScan注解）
    @Bean
    public ServletRegistrationBean getFirstServlet(){
        ServletRegistrationBean servRegBean = new ServletRegistrationBean();
        servRegBean.setServlet(new FirstServlet());
//        List<String> urlMappings = new ArrayList<>();
//        urlMappings.add("/first");
        servRegBean.addUrlMappings("/first", "/firstServlet");  //访问Url,可以添加多个
        servRegBean.setLoadOnStartup(1); //设置加载顺序
        return servRegBean;
    }

    //注册过滤器
    @Bean
    public FilterRegistrationBean getFirstFilter(){
        FilterRegistrationBean filterRegBean = new FilterRegistrationBean();
        filterRegBean.setFilter(new FirstFilter());
        List<String> urlPatterns = new ArrayList<>();
        urlPatterns.add("/*"); //拦截路径，可以添加多个
        filterRegBean.setUrlPatterns(urlPatterns);
        filterRegBean.setOrder(1); //设置注册顺序
        return filterRegBean;
    }

    //注册监听器
    @Bean
    public ServletListenerRegistrationBean<ServletContextListener> getFirstListener(){
        ServletListenerRegistrationBean listenRegBean = new ServletListenerRegistrationBean();
        listenRegBean.setListener(new FirstListenter());
        listenRegBean.setOrder(1);
        return listenRegBean;
    }
}
```

2. 采用注解方式注册

核心注解 @ServletComponentScan, 扫码@WebServlet、@WebFilter、@WebListener

自定义Servlet
```java
@WebServlet(urlPatterns = "/second")
public class SecondServlet extends HttpServlet {

    //处理get请求
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        System.out.println("SecondServlet 执行doGet");
        doPost(req, resp);
    }

    //处理post请求
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        System.out.println("SecondServlet 执行doPost");
        PrintWriter out = resp.getWriter();
        out.println("SecondServlet outer");
    }

    @Override
    public void init(ServletConfig config) throws ServletException {
        System.out.println("SecondServlet init");
        super.init(config);
    }

    @Override
    public void destroy(){
        System.out.println("SecondServlet destroy");
        super.destroy();
    }
}
```
自定义Filter
```java
@WebFilter(filterName = "secondFilter", urlPatterns = "/**")
public class SecondFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("SecondFilter");
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("SecondFilter init");
    }

    @Override
    public void destroy() {
        System.out.println("SecondFilter destroy");
    }
}
```

自定义Listener
```java
@WebListener
@WebListener
public class SecondListenter implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("SecondListenter 正在初始化");
        System.out.println("servlet container:" + sce.getServletContext().getServerInfo());
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("SecondListenter正在销毁");
    }
}
```

修改启动类，添加@ServletComponentScan注解
```java
@ServletComponentScan //启动时会扫描@WebServlet、@WebFilter和@WebListener注解，并创建该类的实例
@SpringBootApplication
public class CaochDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(CaochDemoApplication.class, args);
    }
}
```

## 3.4 配置拦截器

拦截器主要用来拦截请求，在请求执行前后进行某些操作（如入参校验和权限验证等）。SringBoot应用中可以存在多个不同的拦截器，根据声明顺序调用执行。

```java
public class FirstHandlerInterceptor implements HandlerInterceptor {

    //在执行处理程序（action）之前调用
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //默认返回true，如果返回false后续拦截器都会失效，但当前拦截器的afterCompletion会继续执行完。
        System.out.println("FirstHandlerInterceptor 正在执行preHandle. 请求路径：" + request.getRequestURI());
        return true;
    }

    //在执行处理程序（action）之后，但在呈现视图之前调用
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
        System.out.println("FirstHandlerInterceptor 正在执行postHandle. 请求路径：" + request.getRequestURI());
    }

    //在执行处理程序（action）之后，呈现试图之后回调
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
        System.out.println("FirstHandlerInterceptor 正在执行afterCompletion. 请求路径：" + request.getRequestURI());
    }
}
```

```java
/**
 * 创建配置类，将拦截器注入容器
 */
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {

    public void addInterceptors(InterceptorRegistry registry) {
        FirstHandlerInterceptor firstHandlerInterceptor = new FirstHandlerInterceptor();
        registry.addInterceptor(firstHandlerInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/free");
        //SecondHandlerInterceptor secondHandlerInterceptor = new SecondHandlerInterceptor();
        //registry.addInterceptor(secondHandlerInterceptor).addPathPatterns("/**");
    }
}
```
对于Spring MVC，最核心的类是DispatcherServlet，所有请求都会通过它分发处理。

## 3.5 配置AOP

Spring AOP的两种方式：
1. JDK动态代理，基于InvocationHanlder接口，反射生成代理接口的匿名类，并调用invoke方法；

2. CGLIB(Code Generation Library)加载并修改字节码文件，并生成子类，覆盖增强其方法；

（其他AOP框架 AspectJ）

（1） AOP术语：
Aspect: 切面

Pointcut: 切点，相当于查询条件

JoinPoint： 连接点

Advice：增强

Weaving：织入

Proxy：AOP织入后的代理类

Target：目标对象，反射后被调用可增强目标类的业务逻辑

Introduction：引入

（2）通知类型，一半按执行顺序：
Around -> Before ->Around -> After -> AfterReturning

定义多个Aspect后执行顺序不可预测，@Order 注解定义顺序

@Pointcut 声明切入点

## 3.6 全局异常处理

Spring Boot 默认通过BasicErrorController类定义的/error映射路径来处理异常信息

### 3.6.1自定义错误页

1. 方式一：

创建文件resources\templates\error.html， 请求不存在，直接跳转到error.html

2. 方式二：

实现ErrorPageRegistrar接口，注册两个错误请求页面

```java
public class MyErrorPageRegistrar implements ErrorPageRegistrar{
    @Override
    public void registerErrorPages(ErrorPageRegistry errPagReg){
        //page400指定路径 /400，page500指定路径 /500 
        //，可以新建一个Controller的action分别映射 /400 和 /500
        errPagReg.addErrorPages(page400, page500);
    }
}
```

新建配置类，生命Bean并注入容器；

```java
@Configuration
public class ErrorCustomConfig{
    @Bean
    public ErrorPageRegistrar errorPageRegistrar(){
        return new MyErrorPageRegistrar();
    }
}
```

### 3.6.2 自定义错误返回

@ControllerAdvice开启全局异常处理
@ExceptinoHandler 声明异常类型如何处理

定义异常处理类
```java
@ControllerAdvice
public class GlobalExeptionHandler{
    //返回页面
    @ExceptionHandler(RuntimeExeption.class)
    public ModelAndView handle(RuntimeException e){
        //...new ModelAndView
        return modelAndView;
    }

    //返回SON
    @ResponseBody
    @ExceptionHandler(MyException.class)
    public T handleMyException(MyException e){
        //...new 
        return tObj;
    }


}
```

## 3.7 静态资源访问

### 3.7.1 默认静态资源访问

SpringBoot默认静态资源访问路径，按优先级顺序：

```path
classpath:META-INF/resources    #Servlet 3.0不允许浏览器直接访问
classpath:resources
classpath:static
classpath:public
```

### 3.7.2 自定义静态资源访问

```prop
#将所有资源重新定位到/resources/**
spring.mvc.static-path-pattern=/resources/**
```

```prop
#自定义静态资源访问路径，支持多个,逗号隔开. 自定义指定后，SpringBoot默认静态资源路径将不再起作用
spring.resources.static-locations=classpath:/mystatic/**,classpath:/mypublic
```

注：继承WebMvcConfigrer目录可指定绝对目录

















