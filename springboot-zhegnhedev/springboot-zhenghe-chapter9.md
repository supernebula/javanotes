# 第9章 Spring Boot 整合消息服务

## 9.1 消息队列

1. AMQP模型
```
发布者[Publisher] -->  【 交换机[Exchange] ==> 队列[Queue] 】  --> 消费者[Consumer]
```

## 9.2 消息中间件之RabbitMQ

2. 交换机类型：

(1)直接交换器（Direct Exchange）

(2)主题交换器(Topic Exchange)

(3)扇形交换器（Fanout Exchange）

(4)头交换器（Headers Exchange）

### 9.2.2 RabbitMQ自动配置

相关类: 
自动配置类:RabbitAutoConfiguration.class

注入的同时，还会注入RabbitTemplate、CachingConnectionFactory、AmqpAdmin。 

配置参数属性类:RabbitProperties.class

配置参数，application.properties

```properties
#服务端口，默认5672
spring.rabbitmq.port=5672
#登录用户名
spring.rabbitmq.username=guest
#登录用户密码
spring.rabbitmq.password=guest
#服务主机
spring.rabbitmq.host=localhost
#连接到RabbitMQ虚拟主机
spring.rabbitmq..virtualHost=/
```


### 9.2.3 RabbitMQ应用案例

简单案例-生产者示例：
```java
public class Sender{
    private final static String HOST_NAME = "1ocalhost";
    private final static String QUEUE_NAME = "demo-queue-mohai";
    public static void main(String[] args) throws Exception {
        final ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost(HOST_NAME);
        try(final Channel channel = connection.createChannel()){
            channel.queueDeclare(QUEUE_NAME, false, false, false, nu11);
            for(int i=0;i<10; i++){
                final String message = "Hello world! The Num="+i;
                System.out.println("Sending the following message to the queue: " + message);
                channel.basicPublish ("", QUEUE_NAME, nu11,message.getBytes("UTF-8"));
            }
        }       
    }
}
```


简单案例-消费者示例：

```java
public class Receiver {
    private final static String HOST_NAME = "localhost";
    private final static String QUEUE_NAME = "demo-queue-mohai";
    public static void main(String[] args) throws Exception{
        final ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory. setHost (HOST_NAME);
        final Connection connection = connectionFactory. newConnection();
        final Channel channel = connection.createChanne1();
        channel.queueDeclare(QUEUE_NAME, false, false, false, nu1l);
        final DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            final String message = new String(delivery,getBody() , "UTF-8");
            System.out.print1n("Received from message from the queue: "+ message);
        };
        channel.basicConsume(QUEUE_NAME, true, deliverCallback,consumerTag ->{});
    }
}
```

引入pom.xml依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

配置参数 application.properties

```properties
#RabbitMQ配置
spring.rabbitmq.hsot=localhost
spring.rabbitmg.post=5672
spring.rabbitmq.username=guest
spring.rabbitmg.password=guest
spring.rabbitmg.publisher-confirms=true
#是否启用发布确认
spring.rabbitmg.publisher-returns=true
#是否启用发布返回
spring.rabbitmq.virtual-host=/
连接Broker的虚拟主机名
#指定client连接的Server地址，多个地址间以逗号分隔（优先取addresses,
spring.rabbitmg.addresses=localhost
然后再取ho3t)
#指定心跳超时时间，单位为S,0为不指定，默认为60s
spring.rabbitmg.requested-heartbeat=60
#指定连接超时时间，单位为ms,0表示无穷大，不超时
spring.rabbitmg.connection-timeout=0
spring.rabbitmg.ssl.enabled=false
#是否支持SSL
spring.rabbitmq.listener.simple.concurrency=10
#最小的消费者数量
spring.rabbitmg.listener.simple.max-concurrency=20
#最大的消费者数量
#指定一个请求能处理多少个消息，如果有事务的话，必须大于等于
BatchSize的数量
spring.rabbitmg.listener.simple.prefetch=5
#指定一个事务处理消息的数量，最好是小于等于prefetch的数量
spring.rabbitmg.listener,simple.batch-size=3
```

新建配置类 config/RabbitDirectConfig.java

```java
package com.mohai.one.springbootrabbitmq.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


/**
 * 配置默认交换器（direct）实现
 * Direct策略（只转发给routingKey相匹配的用户）
 * @Auther: moerhai@qq.com
 * @Date: 2020/10/4 11:30
 */
@Configuration
public class RabbitDirectConfig {

    public final static String DIRECT_NAME = "amqp-direct";

    public final static String QUEUE_NAME = "my-queue";

    public final static String ROUTING_KEY = "my-routing";

    /**
     * 创建DirectExchange对象
     * @return
     */
    @Bean
    DirectExchange directExchange (){
        // 第一个参数是交换器名称、
        // 第二个参数是否持久化，重启后是否依然有效、
        // 第三个参数是长期未使用时是否自动删除
        return new DirectExchange (DIRECT_NAME,false,false);
    }

    /**
     * 提供一个消息队列Queue
     * @return
     */
    @Bean
    Queue queue(){
        // 第一个参数name值为队列名称，routingKey会与它进行匹配
        // 第二个参数durable是否持久化
        return new Queue(QUEUE_NAME,false);
    }

    /**
     * 创建一个Binding对象将Exchange和Queue绑定再一起
     * @return
     */
    @Bean
    Binding binding(){
        // 将队列queue和DirectExchange绑定在一起
        return BindingBuilder.bind(queue()).to(directExchange()).with(ROUTING_KEY);
    }

}

```

新建消息发送类 sender/MessageSender.java

```java
package com.mohai.one.springbootrabbitmq.sender;

import com.mohai.one.springbootrabbitmq.config.RabbitDirectConfig;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.UUID;

/**
 * 消息生产者 发送消息
 * @Auther: moerhai@qq.com
 * @Date: 2020/10/4 11:57
 */
@Component
public class MessageSender {

    @Autowired
    RabbitTemplate rabbitTemplate;

    /**
     * 发送消息
     * @param info
     */
    public void send(String info){
        System.out.println("发送消息>>>"+info);
        MessageProperties messageProperties = new MessageProperties();
        // 构建消息属性，设置消息id
        messageProperties.setMessageId(UUID.randomUUID().toString());
        // 构建消息，设置消息内容
        Message message = new Message(info.getBytes(),messageProperties);
        rabbitTemplate.convertAndSend(RabbitDirectConfig.DIRECT_NAME,RabbitDirectConfig.ROUTING_KEY,message);
    }

}
```

新建消息发送类 receiver/DirectReceiver.java
```java
package com.mohai.one.springbootrabbitmq.receiver;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.*;
import org.springframework.stereotype.Component;

/**
 * 配置消息消费者 接收消息
 * @Auther: moerhai@qq.com
 * @Date: 2020/10/4 11:33
 */
@Component
public class DirectReceiver {

    // 处理接收到的消息
    // 注意参数需要消息发送者发送的消息的类型保持一致
    @RabbitHandler
    //监听queues内的队列名称列表
    @RabbitListener(queues = {"my-queue"})
    public void handler(Message message) {
        System.out.println("接收消息>>>"+new String(message.getBody()));
    }

    @RabbitListener(bindings = {@QueueBinding(value = @Queue("my-queue"), exchange = @Exchange("my-exchange"), key = {"my-routing-key"})})
    public void receiveMsg2(String message){
        System.out.println("接收消息>>>"+ message);
        //todo json反序列化， 业务逻辑等
    }

}
```

新建消息测试类 controller/IndexController.java

```java
package com.mohai.one.springbootrabbitmq.controller;

import com.mohai.one.springbootrabbitmq.sender.MessageSender;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Auther: moerhai@qq.com
 * @Date: 2020/10/4 11:34
 */
@RestController
public class IndexController {

    @Autowired
    MessageSender messageSender;

    @RequestMapping("/index")
    public String index(){
        messageSender.send("hello world!");
        return "SUCCESS";
    }

}
```


## 9.4 消息中间件之Kafka

Kafka 高性能、跨语言、分布式式发布订阅消息队列系统，理念不同，不支持AMQP。下载kafka [http://kafka.apache.rog](http://kafka.apache.rog)

注：使用Kafka会有消息重复消费的问题.

### 9.4.1 kafka的基本概念

主题（topic）和 日志（Log），topic表示主题，是发布消息的类别名，可以用来区分业务系统。一个topic可以有0~N个消费者。每个Topic在kafka集群中维护一个分区日志。

### 9.4.2 Kafka自动配置

在springboot中，kafka 相关的类，KafkaAutoConfiguration（自动配置类中初始化了KafkaTemplate、ConsumerFactory、ProducerFactory、KafkaTranactionManager）。 KafkaProperties是其配置类。


### 9.4.3 Kafka应用案例

1. 使用Kafka的测试组件，使用@EnbeddedKafka，引入pom.xml依赖。



## 9.5消息中间件RocketMQ

### 9.5.1 RocketMQ的基本概念

RocketMQ中基本的消息模型主要由Producer、Broker、Consumer三部分组成。Producer负责生产消息，由业务系统创建，并把消费发送到Broker服务器。

RocketMQ提供多种发送方式：同步发送、异步发送、顺序发送、单项发送等。同步和异步发送需要Broker服务器返回确认消息，其他不需要。

官方提供的部署方案中默认给出3中方式：2m-noslave(双主无从模式)、2m-2s-async（双主双从,异步复制模式）、2m-2s-sync（双主双从，同步双写模式）。

Broker存储多个Topic消息；Message Queue用于存储消息的物理地址；ConsumerGroup消费者组即同一类Consumer示例的集合；ProducerGroup生产者组，即同一类Producer同一类消息；

RocketMQ在4.3.0及以后支持分布式事务消息（才用2PC思想提交事务消息，同时增加了一个补偿逻辑处理二阶段提交或者失败消息）。

### 9.5.2 RocketMQ自动配置

在SpringBoot中使用RocketMQ, pom.xml引入依赖：

```xml
<!--引入RocketMQ开始-->
 <dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>4.7.1</version>
</dependency>

<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.7.1</version>
</dependency>
<!--引入RocketMQ结束-->
```

#### 注解

1. 配置客户端消息监听器注解 @RocketMQMessageListener
使用案例如下：

2. 配置本地事务监听器 @RocketMQTransactionListener。其需要自定义配置类，实现RocketMQLocalTransactionListener接口，并使用@RocketMQTransactionListener注解标记。


### 9.5.3 RocketMQ应用案例

配置系统环境变量ROCKETMQ_HOME

启用rocketmq
```shell
start mqnamesrv.cmd
start mqbroker.cmd -n 127.0.0.1:9876
```

Rocketmq的官方控制台 RocketMQ-Console，下载 https://codeload.github.com/apache/rocketmq-externals/zip/master 。
进入rocketmq-externals-master\rocketmq-console目录下，找到application.properties文件，修改server.port参数（默认为8080），避免冲突，改掉。
重新进入rocektmqconsole，从新打包：
```shell
mvn clean package -DskipTests
```

1. 在SpringBoot中pom.xml引入依赖：

```xml
<!--引入RocketMQ开始-->
 <dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>4.7.1</version>
</dependency>

<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.7.1</version>
</dependency>
<!--引入RocketMQ结束-->
```

在项目中新建两个模块，分别是生产者模块和消费者模块，都引入以上RocketMQ的相关依赖。

2. 以下消费者模块中的配置，application.properties

```properties
server.port=8081
spring.application.name=rocketmq-consume-demo
#消费者相关配置
rocketmq.name-server=localhost:9876
#自定义主题名称
evol.rocketmq.topic=test-topic-1
evol.rocektmq.msgExtTopic=test-message-ext-topic
```

3. 编写监听器



### 安装RocketMQ 和 管理工具 rocketmq-dashboard

下载地址（包含安装步骤和命令）：

https://rocketmq.apache.org/docs/quick-start/

https://github.com/apache/rocketmq-dashboard

启动后







#### SpringBoot集成RocketMQ案例



##### 安装


下载：https://rocketmq.apache.org/docs/quick-start/

按照官网安装rocketmq， 修改/rocketmq-4.9.4/conf/2m-noslave/broker-a.properties 文件，组件以下内容：
```properties
#rocketmq部署的服务IP
namesrvAddr=192.168.0.39:9876
```
配置完成。

注：macos安装会启动失败，centos7安装没这个问题。

下载 ： https://github.com/apache/rocketmq-dashboard

按照官网步骤源码安装 rocketmq-dashboard， 修改其端口为9091 （默认的8080容易被占用）。
浏览器访问 http://192.168.0.39:9091/， NameServerAddressList添加rocketmq节点 192.168.0.39:9876
，配置完成。




##### 实例代码

参考： https://cloud.tencent.com/developer/article/2000243


1. 引入依赖pom.xml
```xml
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-spring-boot-starter</artifactId>
            <version>2.2.2</version>
        </dependency>
```



2. 配置属性配置application.properties
```properties
server.port=8082
spring.application.name=coach-demo-mq
#rocketmq相关配置，注意rocketmq-spring-boot-starter 2.2以上是nameServer （2.2之前是name-server）
rocketmq.nameServer=192.168.0.39:9876
rocketmq.producer.group=my-group
```

注：如果属性设置错误，项目会启动失败，报错：".........RocketMQTemplate‘ that could not be found.

3. 编写发送、接受、DTO类

com.evol.rocketmq.DemoOrderDto， DTO类

```java
@Data
@AllArgsConstructor
public class DemoOrderDto implements Serializable {

    private String orderNo;

    private Integer amount;

    private Date createTime;
}
```

定义RocketMQ的TOPIC，放在常量里 com.evol.rocketmq.TopicConstant
```java
public class TopicConstant {
    public static final String DEMO_TOPIC1 = "demo-topic1";
}
```

com.evol.rocketmq.RocketMQReceiver, 消费者类
```java
@Component
@RocketMQMessageListener(topic = TopicConstant.DEMO_TOPIC1, //topic主题
        consumerGroup = "demo-group1",          //消费组
        messageModel = MessageModel.CLUSTERING,
        consumeMode = ConsumeMode.ORDERLY)
@Slf4j
public class RocketMQReceiver implements RocketMQListener<String> {

    @Override
    public void onMessage(String message) {
        log.info("接受到消息：{}", message);
    }
}
```

com.evol.rocketmq.RocketMQSender,消息发送(生产)者
```java
@Component
@Slf4j
public class RocketMQSender {

    @Autowired
    RocketMQTemplate rocketMQTemplate;

    /**
     *
     * @param topic @link TopicConstant.DEMO_TOPIC1
     * @param message
     */
    public void sendMessage(String topic, String message){
        SendResult result = rocketMQTemplate.syncSend(topic, message);
        log.info("发送结果：{}", JSON.toJSONString(result));
    }

    public <T extends Serializable> SendStatus sendDto(String topic, T dto){
        SendResult result = rocketMQTemplate.syncSend(topic, JSON.toJSONString(dto));
        log.info("发送结果：{}", JSON.toJSONString(result));
        return result.getSendStatus();
    }
}
```

com.evol.controller.DemoMQController , 测试用的controller，访问action发送消息

```java
@RestController
public class DemoMQController {

    @Autowired
    private RocketMQSender rocketMQSender;

    @GetMapping("/testSendMsg")
    public String testSendMessage(@RequestParam(required = false) String msg){
        if(StringUtils.isBlank(msg)){
            msg = "Hello World";
        }
        rocketMQSender.sendMessage(TopicConstant.DEMO_TOPIC1, msg);
        return "ok";
    }

    @GetMapping("/testSendObj")
    public String testSendDto(){
        DemoOrderDto demoOrderDto = new DemoOrderDto("OR202204051220", 100, new Date());
        Object result = rocketMQSender.sendDto(TopicConstant.DEMO_TOPIC1, demoOrderDto);
        return "ok" + result;
    }
}

```

浏览器访问测试：
http://localhost:8082/testSendObj

http://localhost:8082/testSendMsg

测试成功。


4. 扩展发送和消费待tag的消息

发送方法：rocketMQTemplate.syncSend(destination, Object message)，destination的格式是 "topic:tag"

案例代码，com.evol.rocketmq.RocketMQSender
```java
@Component
@Slf4j
public class RocketMQSender {
    public void sendTagMessage(String topic, String tag, String message){
        SendResult result = rocketMQTemplate.syncSend(topic + ":" + tag, message);
        log.info("发送结果：{}", JSON.toJSONString(result));
    }
}
```

```java

@RestController
public class DemoMQController {
    @GetMapping("/testSendTagMsg")
    public String testSendTagMessage(@RequestParam(required = false) String msg){
        if(StringUtils.isBlank(msg)){
            msg = "Hello World";
        }
        rocketMQSender.sendTagMessage(TopicConstant.DEMO_TOPIC1, "tag1", msg);
        return "ok";
    }
}
```

消费代码，注解@RocketMQMessageListener的 selectorExpression生命tag。实例代码如下：

```java
    @Service
    @RocketMQMessageListener(topic = TopicConstant.DEMO_TOPIC1, consumerGroup = "demo-group1", selectorExpression =
            "tag1 || tag2")
    public class RocketMqUpdateBalanceEventConsumer2 implements RocketMQListener<String>{


        /**
         * 封装过，无异常会自动ack，有异常mq重发
         * @param msg
         */
        @SneakyThrows
        @Override
        public void onMessage(String msg) {
            System.out.println("接受到消息为:RocketMqUpdateBalanceEventConsumer2:" + msg);

            //todo 关于orderCancelParam的业务逻辑；
        }
    }
```







