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

@Pointcut 声明切入点，定义切入点的9中方法，execute表达式：

```java
@Pointcut("execution(public * com.evol.buniness.*.*(..))")
```

拦截任意公共方法
```java
@Pointcut("execution(public * *(..))")
```
拦截以set开头的任意方法
```java
@Pointcut("execution(* set*(..))")
```

拦截类AccountService或者接口中的方法
```java
@Pointcut("execution(* com.xyz.service.AccountService.*(..))")
```

拦截包中定义的方法，不包含子包中的方法
```java
@Pointcut("execution(* com.xyz.service.*.*(..))")
```

拦截包或者子包中定义的方法
```java
@Pointcut("execution(* com.xyz.service..*.*(..))")
```

[...更多](https://www.cnblogs.com/itsoku123/p/10744244.html)


实战，为controller 定义切面

定义注解类
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Log {
    //操作名称
    String name() default "";
}
```

定义切面类
```java
//定义切面，@Aspect使之成为切面类
@Aspect
@Component
public class AopAspect {

    //定义切入点，切入点为标com.evol.buniness包及其子包的，通过@Pointcut生命切入点表达式
    //@Pointcut("execution(public * com.evol.buniness.*.*(..))")
    //定义切入点，切入点为标有注解@Log的所有函数，通过@Pointcut生命切入点表达式
    @Pointcut("@annotation(com.evol.aop.annotation.Log)")
    public void LogAspect(){
    }

    //在连接点执行之前执行的通知
    @Before("LogAspect()")
    public void doBeforeLog(){
        System.out.println("执行controller前置通知");
    }

    //使用环绕通知，注意该方法需要返回值
    @Around("LogAspect()")
    public Object doAroundLog(ProceedingJoinPoint pjp) throws  Throwable{
        try{
            System.out.println("开始执行contoller环绕通知");
            Object obj = pjp.proceed();
            System.out.println("结束执行controller环绕通知");
            return obj;
        }catch (Throwable e){
            System.out.println("出现异常");
            throw e;
        }
    }

    //在连接点执行结束之后执行的通知
    @After("LogAspect()")
    public void doAfterLog(){
        System.out.println("执行Controller后置结束通知");
    }

    //在连接点执行结束并返回之后执行的通知
    @AfterReturning("LogAspect()")
    public void doAfterReturnLog(JoinPoint joinPoint){
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        Log log = method.getAnnotation(Log.class);
        String name = log.name();
        System.out.println(name);
        System.out.println("执行controller后置返回通知");
    }

    @AfterThrowing(pointcut = "LogAspect()", throwing = "e")
    public void doAfterThrowingLog(JoinPoint joinPoint, Throwable e) {
        System.out.println("=======异步通知开始======");
        System.out.println("异步代码：" + e.getClass().getName());
        System.out.println("异步信息：" + e.getMessage());
    }
}
```

```java
@RestController
public class TestController {
        @Log(name = "访问/free接口")
    @GetMapping("/free")
    public String free(){
        return "排查在拦截器Interceptor之外，不会被拦截";
    }
}
```

访问http://localhost:8085/free
控制台打印日志如下：
```log
开始执行contoller环绕通知
执行controller前置通知
结束执行controller环绕通知
执行Controller后置结束通知
访问/free接口
执行controller后置返回通知
```

## 3.6 全局异常处理

Spring Boot 默认通过BasicErrorController类定义的/error映射路径来处理异常信息

### 3.6.1自定义错误页

1. 方式一：

创建文件resources\templates\error.html， 请求不存在，直接跳转到error.html

创建Controller
```java
@Controller
public class ExTestController {
    @GetMapping("/error")
    public String error(){
        return "error";
    }
}
```

2. 方式二：

实现ErrorPageRegistrar接口，注册两个错误请求页面

```java
public class MyErrorPageRegistrar implements ErrorPageRegistrar{
    @Override
    public void registerErrorPages(ErrorPageRegistry errPagReg){
        ErrorPage page400 = new ErrorPage(HttpStatus.BAD_REQUEST, "/400");
        ErrorPage page500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/500");
        //page400指定路径 /400，page500指定路径 /500
        //，可以新建一个Controller的action分别映射 /400 和 /500
        errPagReg.addErrorPages(page400, page500);
    }
}
```

声明配置类，注入容器
```java
@Configuration
public class ErrorCustomConfig {
    @Bean
    public ErrorPageRegistrar errorPageRegistrar(){
        return new MyErrorPageRegistrar();
    }
}
```
创建测试方法，并请求
```java
@Controller
public class ExTestController {
    @GetMapping("/400")
    public String error400(){
        return "400";
    }

    @GetMapping("/500")
    public String error500(){
        return "500";
    }

    @GetMapping("/ex/index")
    @ResponseBody
    public String exIndex(){
        int n =1 /0;
        return "SUCCESS";
    }
}
```

请求 localhost:8085/ex/index
返回500.html内容：“500 error页面”

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
public class GlobalExceptionHandler {
    //返回错误页面
    @ExceptionHandler(RuntimeException.class)
    public ModelAndView handle(RuntimeException ex){
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.setViewName("errorPage");
        modelAndView.addObject("code", 500);
        modelAndView.addObject("msg", ex.getMessage());
        return modelAndView;
    }

    //返回JSON
    @ResponseBody
    @ExceptionHandler(MyException.class)
    public Map<String, Object> handleMyException(MyException ex){
        Map<String, Object> map = new HashMap<>();
        map.put("code", 500);
        map.put("message", ex.getMsg());
        return map;
    }
}
```

定义异常类
```java
@Data
public class MyException extends RuntimeException{
    private int code;
    private String msg;
    public MyException(String msg){
        super(msg);
        this.code = 500;
        this.msg = msg;
    }
}
```

定义测试Controller
```java
 @GetMapping("/excustom")
    @ResponseBody
    public String exCustom(){
        throw new RuntimeException("excustom1");
    }

    @GetMapping("/excustomp")
    public String exCustomPage(){
        if(1 ==1 ){
            throw new MyException("excustom1");
        }
        return "error";
    }
```

分别访问如下url，测试完成
http://localhost:8085/excustom
http://localhost:8085/excustomp





## 3.7 静态资源访问

### 3.7.1 默认静态资源访问

SpringBoot默认静态资源访问路径，按优先级顺序：

```path
classpath:META-INF/resources    #Servlet 3.0不允许浏览器直接访问
classpath:resources
classpath:resources/static/1.html
classpath:resources/public
```

访问http://localhost:8085/1.html

### 3.7.2 自定义静态资源访问

这条设置不太好用
```application.properties
#将所有资源重新定位到/resources/**
spring.mvc.static-path-pattern=/resources/**
```

```application.properties
#自定义静态资源访问路径，支持多个,逗号隔开. 自定义指定后，SpringBoot默认静态资源路径将不再起作用
spring.resources.static-locations=classpath:/mystatic,classpath:/mypublic
```
自定义为mystatic后，访问不在定位到static目录，而是mystatic目录

注：继承WebMvcConfigrer目录可指定绝对目录

## 3.8 配置CORS实现跨域

条件：必须浏览器和服务器同时支持跨域功能。
三种方式：

1. 实现WebMvcConfigurer接口

```java
/**
 * CORS方式一
 * spring-webmvc-5.2.0R
 * springboot2.2.0R之前的版本，这种方式弊端，如果自定义拦截器，跨域配置会失效。执行顺序先处理拦截器，在执行请求映射逻辑
 */
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")  //指定可以跨域的路径
                .allowedHeaders("*")           //服务器允许的请求头
                .allowedMethods("POST", "PUT", "GET", "OPTIONS", "DELETE") //服务器允许的请求方法
                .allowCredentials(true)         //允许带cookie的跨域请求Access-Control-Allow-Credentials
                .allowedOrigins("*")            //"*" 表示服务端允许所有的访问请求
                .maxAge(3600);                  //预检请求的缓存时间，单位s，默认1800s
    }
}
```

2. 使用CoreFilter过滤器完美解决跨域问题，实现全局跨域。

```java
/**
 * 方式二 CorsFilter过滤器,Springboot 没有自动设置，需要手动注讲CorsFilter注入容器。
 * 过滤器先于拦截器执行
 */
@Configuration
public class CorsFilterConfig {
    @Bean
    public FilterRegistrationBean<CorsFilter> coreFilter(){
        FilterRegistrationBean<CorsFilter> corsFilterFilterRegistrationBean = new FilterRegistrationBean<>();
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        //服务端允许的请求头
        corsConfiguration.addAllowedHeader("*");
        //服务端允许的域请求来源
        corsConfiguration.addAllowedOrigin("*");
        //服务器允许的请求方法
        corsConfiguration.setAllowedMethods(Arrays.asList("POST", "PUT", "GET", "OPTIONS", "DELETE"));
        //允许带cookie的跨域请求Access-Control-Allow-Credentials
        corsConfiguration.setAllowCredentials(true);
        //预检请求的客户端缓存时间，单位s，默认1800s
        corsConfiguration.setMaxAge(3600L);
        //指定被跨域的路径
        source.registerCorsConfiguration("/**", corsConfiguration);
        //设置加载顺序为-1，该值越小优先级越高
        corsFilterFilterRegistrationBean.setOrder(-1);
        return corsFilterFilterRegistrationBean;
    }
}
```

3. 使用@CrossOrigin注解

```java
@RestController
public class CorsTestIndexController {
    /**
     * @CrossOrigin 默认允许所有的访问请求，允许客户端所有请求头，预响应最大缓存时间1800s
     * @return
     */
    @CrossOrigin
    //在注解中配置参数，不常用
    //@CrossOrigin(origins = {""}, allowedHeaders = {"GET", "POST"}, exposedHeaders = {""}, methods = {RequestMethod.GET, RequestMethod.POST}, allowCredentials = "cookie", maxAge = 1800)
    @RequestMapping("/corsindex")
    public String index(){
        return "hello man";
    }
}
```

## 3.9 文件上传

Springboot模式支持多文件上传。如果需要，还可更换为apache Commons FileUpload包

```java
/**
 * 如果需要，更换为apache Commons FileUpload包
 */
@Configuration
public class FileUploadConfig{
    @Bean
    public MultipartResolver multipartResolver() throws IOException{
        CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();
        return multipartResolver;
    }
}
```

文件上传controller
```java
@RestController
public class FileUploadController{

    //单文件上传
    @PostMapping(value ="/upload")
    public String upload(@RequestParam("file") MultipartFile file, @RequestParam("description") String description) throws Exception {
        if (file == null || file.isEmpty()) {
            return "文件为空";
        }
        //获取文件名
        String fileName = file.getOriginalFilename();
        System.out.println("文件名称："+fileName);  //打印文件上传名称
        System.out.println("文件描述："+description); //打印文件上传名称
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
        String subPath = sdf.format (new Date());
        String basePath = subPath +"/" + fileName;
        System.out.println("保存文件路径："+basePath);
        File dest = new File(basePath);
        //检测是否存在目录
        if (!dest.getParentFile().exists()){
            //新建文件夹
            dest.getParentFile().mkdirs();
        }
        //文件保存
        file.transferTo(dest);
        return "SUCCESS";
    }

    //对于多个文件上传来说，只需要在方法中传入MultipartFile[数组即可，或者通过MultipartHttpServletRequest#getFiles("file")方法来获取上传的多个文件。
    //我们直接来看代码示例，相信读者会看到熟悉的代码。
    @PostMapping ("/uploads")
    public String uploads(MultipartFile[]uploadFiles, HttpServletRequest request){
    List<MultipartFile> files =((MultipartHttpServletRequest)request).getFiles("file");
        String realPath=request.getSession().getServletContext().getRealPath("/uploadFile/");
        System.out.println(realPath);
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
        String subPath = sdf.format (new Date());
        for (MultipartFile uploadFile : uploadFiles){
            File folder = new File(realPath + subPath);
            if (folder.isDirectory()){
                folder.mkdirs();
                String oldName = uploadFile.getOriginalFilename();
                try{
                    uploadFile.transferTo(new File(folder,oldName));
                }catch (IOException e){
                        e.printStackTrace();
                }
            }
        }
        return "SUCCESS";
    }
}
```

## 3.10 配置启动加载项

SpringBoot 启动后提供的执行任务的扩展功能，并在SpringApplication#run()方法执行结束之前调用的。@Order注解设定运行顺序

定义 MyCommandLineRunner
```java
@Component
@Order(1)
public class MyCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String ... arg) throws Exception{
        System.out.println("MyCommandLineRunner正在执行run");
    }
}
```

```java
@Component
@Order(0)
public class MyApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("MyApplicationRunner正在执行run");
    }
}
```

```java
@SpringBootApplication
public class CaochDemoApplication {
    public static void main(String[] args) {
        System.out.println("SpringBootApplication 运行run之前");
        SpringApplication.run(CaochDemoApplication.class, args);
        System.out.println("SpringBootApplication 运行run之后");
    }
}
```
启动后，打印日志如下：
```log
SpringBootApplication 运行run之前
MyApplicationRunner正在执行run
MyCommandLineRunner正在执行run
SpringBootApplication 运行run之后
```

## 3.11 配置日志

### 3.11.1 Log4J 2配置

Log4j2 是高效\低延时的异步处理框架，通过“生产者-消费者”模式实现异步记录；
Log4j2 异步日志性能优于Log4j、LogBack和Logging吞吐量的18倍；

springboot启用log4j2，修改pom.xml文件
```xml
        <!--引入log4j 2依赖 开始-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
        <!--引入log4j 2依赖 结束-->
```

在resources目录下新建xml配置文件，支持log4j2.xml和log4j2-spring.xml; 不在支持以 .properties后缀的配置文件。如果自定义日志配置文件名和路径，在application.properties配置如下：
```prop
logging.config=classpath:log4j2.xml
```
log4j2.xml配置内容如下：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<Configuration status="INFO">
    <Appenders>
        <Console name="CONSOLE" target="SYSTEM_OUT">
            <PatternLayout charset="UTF-8" pattern="$d(yyyy-MM-dd HH:mm:ss,SSS}%5 %c{1}:%L - %m%n"/>
        </Console>
        <File name="File" fileName="/home/logs/springboot.log">
            <PatternLayout pattern="[%-5p] %d %c - %m%n"/>
        </File>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="CONSOLE"/>
            <AppenderRef ref="File"/>
        </Root>
    </Loggers>
</Configuration>
```

### 3.11.2 Logback配置

Logback是SpringBoot默认的日志框架，日志级别ERROR、WARN、INFO、DEBUG、TRACE。SpringBoot只会输出ERROR、WARN、INFO级别。

引入依赖pom.xml
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

在resources目录下添加配置文件logback-spring.xml

在application.properties如下配置：
```prop
logback.path.application=/home/logs/mohai
logback.loglevel=INFO
```

 






















