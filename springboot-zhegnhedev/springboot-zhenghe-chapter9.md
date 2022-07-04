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

<de>




