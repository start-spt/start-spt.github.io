---
title: 使用rabbitMQ实现聊天消息的转发
date: 2022-06-18 14:15:01
tags:
 - MQ
categories:
 - 分布式
cover: https://s2.loli.net/2022/06/29/Bta5HYLQzPxb4Uo.jpg
---

# 项目结构
## 项目结构图
![2022-06-20_pro](./rabbitMQ/2022-06-20_pro.png)
## 父pom文件
``` pom
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.spt</groupId>
    <artifactId>cloud_mq</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>mq_rabbit</module>
        <module>mq_rabbit02</module>
        <module>mq_rabbit03_spring</module>
    </modules>

    <name>cloud_mq</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
    </properties>

    <!--  因为是总项目 所以用dependencyManagement来管理  因为其他的子项目就不会来管理版本了了 可以直接引用 -->
    <dependencyManagement>
        <dependencies>
            <!-- springboot的依赖-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.1.4.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>


```
## 子模块结构图
![2022-06-20_model](./rabbitMQ/2022-06-20_model.png)
## 子模块pom文件
```pom
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <artifactId>cloud_mq</artifactId>
    <groupId>com.spt</groupId>
    <version>1.0-SNAPSHOT</version>
  </parent>

  <artifactId>mq_rabbit</artifactId>
  <name>mq_rabbit</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>

  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>

```
## 子模块配置文件
```yml
server:
  port: 8001

spring:
  application:
    name: spirng-boot-rabbitmq
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    password: admin
    username: admin
    virtualHost: /
```
### 参数解释
 
 - **host** : rabbitMQ服务的地址
 - **port** : rabbitMQ服务的端口 (5672为mq应用访问端口,15672为web访问端口)
 - **password** : 密码
 - **username** : 用户名
 - **virtualHost** : 虚拟消息服务器 (每个VirtualHost相当于一个相对独立的RabbitMQ服务器；每个VirtualHost之间是相互隔离的，exchange、queue、message不能互通。 )

### RabbitMQ配置参数详解:
```properties
#基础信息
spring.rabbitmq.host: 默认localhost
spring.rabbitmq.port: 默认5672
spring.rabbitmq.username: 用户名
spring.rabbitmq.password: 密码
spring.rabbitmq.virtual-host: 连接到代理时用的虚拟主机
spring.rabbitmq.addresses: 连接到server的地址列表（以逗号分隔），先addresses后host 
spring.rabbitmq.requested-heartbeat: 请求心跳超时时间，0为不指定，如果不指定时间单位默认为妙
spring.rabbitmq.publisher-confirms: 是否启用【发布确认】，默认false
spring.rabbitmq.publisher-returns: 是否启用【发布返回】，默认false
spring.rabbitmq.connection-timeout: 连接超时时间，单位毫秒，0表示永不超时 

#SSL
spring.rabbitmq.ssl.enabled: 是否支持ssl，默认false
spring.rabbitmq.ssl.key-store: 持有SSL certificate的key store的路径
spring.rabbitmq.ssl.key-store-password: 访问key store的密码
spring.rabbitmq.ssl.trust-store: 持有SSL certificates的Trust store
spring.rabbitmq.ssl.trust-store-password: 访问trust store的密码
spring.rabbitmq.ssl.trust-store-type=JKS：Trust store 类型.
spring.rabbitmq.ssl.algorithm: ssl使用的算法，默认由rabiitClient配置
spring.rabbitmq.ssl.validate-server-certificate=true：是否启用服务端证书验证
spring.rabbitmq.ssl.verify-hostname=true 是否启用主机验证

#缓存cache
spring.rabbitmq.cache.channel.size: 缓存中保持的channel数量
spring.rabbitmq.cache.channel.checkout-timeout: 当缓存数量被设置时，从缓存中获取一个channel的超时时间，单位毫秒；如果为0，则总是创建一个新channel
spring.rabbitmq.cache.connection.size: 缓存的channel数，只有是CONNECTION模式时生效
spring.rabbitmq.cache.connection.mode=channel: 连接工厂缓存模式：channel 和 connection

#Listener
spring.rabbitmq.listener.type=simple: 容器类型.simple或direct
 
spring.rabbitmq.listener.simple.auto-startup=true: 是否启动时自动启动容器
spring.rabbitmq.listener.simple.acknowledge-mode: 表示消息确认方式，其有三种配置方式，分别是none、manual和auto；默认auto
spring.rabbitmq.listener.simple.concurrency: 最小的消费者数量
spring.rabbitmq.listener.simple.max-concurrency: 最大的消费者数量
spring.rabbitmq.listener.simple.prefetch: 一个消费者最多可处理的nack消息数量，如果有事务的话，必须大于等于transaction数量.
spring.rabbitmq.listener.simple.transaction-size: 当ack模式为auto时，一个事务（ack间）处理的消息数量，最好是小于等于prefetch的数量.若大于prefetch， 则prefetch将增加到这个值
spring.rabbitmq.listener.simple.default-requeue-rejected: 决定被拒绝的消息是否重新入队；默认是true（与参数acknowledge-mode有关系）
spring.rabbitmq.listener.simple.missing-queues-fatal=true 若容器声明的队列在代理上不可用，是否失败； 或者运行时一个多多个队列被删除，是否停止容器
spring.rabbitmq.listener.simple.idle-event-interval: 发布空闲容器的时间间隔，单位毫秒
spring.rabbitmq.listener.simple.retry.enabled=false: 监听重试是否可用
spring.rabbitmq.listener.simple.retry.max-attempts=3: 最大重试次数
spring.rabbitmq.listener.simple.retry.max-interval=10000ms: 最大重试时间间隔
spring.rabbitmq.listener.simple.retry.initial-interval=1000ms:第一次和第二次尝试传递消息的时间间隔
spring.rabbitmq.listener.simple.retry.multiplier=1: 应用于上一重试间隔的乘数
spring.rabbitmq.listener.simple.retry.stateless=true: 重试时有状态or无状态
 
spring.rabbitmq.listener.direct.acknowledge-mode= ack模式
spring.rabbitmq.listener.direct.auto-startup=true 是否在启动时自动启动容器
spring.rabbitmq.listener.direct.consumers-per-queue= 每个队列消费者数量.
spring.rabbitmq.listener.direct.default-requeue-rejected= 默认是否将拒绝传送的消息重新入队.
spring.rabbitmq.listener.direct.idle-event-interval= 空闲容器事件发布时间间隔.
spring.rabbitmq.listener.direct.missing-queues-fatal=false若容器声明的队列在代理上不可用，是否失败.
spring.rabbitmq.listener.direct.prefetch= 每个消费者可最大处理的nack消息数量.
spring.rabbitmq.listener.direct.retry.enabled=false  是否启用发布重试机制.
spring.rabbitmq.listener.direct.retry.initial-interval=1000ms # Duration between the first and second attempt to deliver a message.
spring.rabbitmq.listener.direct.retry.max-attempts=3 # Maximum number of attempts to deliver a message.
spring.rabbitmq.listener.direct.retry.max-interval=10000ms # Maximum duration between attempts.
spring.rabbitmq.listener.direct.retry.multiplier=1 # Multiplier to apply to the previous retry interval.
spring.rabbitmq.listener.direct.retry.stateless=true # Whether retries are stateless or stateful.

#Template
spring.rabbitmq.template.mandatory: 启用强制信息；默认false
spring.rabbitmq.template.receive-timeout: receive() 操作的超时时间
spring.rabbitmq.template.reply-timeout: sendAndReceive() 操作的超时时间
spring.rabbitmq.template.retry.enabled=false: 发送重试是否可用 
spring.rabbitmq.template.retry.max-attempts=3: 最大重试次数
spring.rabbitmq.template.retry.initial-interva=1000msl: 第一次和第二次尝试发布或传递消息之间的间隔
spring.rabbitmq.template.retry.multiplier=1: 应用于上一重试间隔的乘数
spring.rabbitmq.template.retry.max-interval=10000: 最大重试时间间隔

```
参考:
1.https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#integration-properties
2.https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/html/common-application-properties.html

 # springBoot整合rabbitMQ
 ## 引入依赖
 ``` pom
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
 ```
 - 父模块引入spring-boot-dependencies确定版本后,子模块不写版本号,会使用默认版本
![2022-06-20_model_mvn](./rabbitMQ/2022-06-20_model_mvn.png)
## 配置MQ
```java
@Configuration
public class RabbitConfig {
    //fanout
    public static final String FANOUT_QUEUE1 = "fanout.queue1";
    public static final String FANOUT_EXCHANGE = "fanout.exchange";

    /**
     * Fanout模式
     * Fanout 就是我们熟悉的广播模式或者订阅模式，给Fanout交换机发送消息，绑定了这个交换机的所有队列都收到这个消息。
     * @return
     */
    @Bean
    public Queue fanoutQueue1() {
        return new Queue(FANOUT_QUEUE1);
    }

    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange(FANOUT_EXCHANGE);
    }

    @Bean
    public Binding fanoutBinding1() {
        return BindingBuilder.bind(fanoutQueue1()).to(fanoutExchange());
    }
}
```
- 这里使用了rabbitMQ的fanout模式:
![2022-06-20_rabbit_fanout](./rabbitMQ/2022-06-20_rabbit_fanout.png)
> **Fanout** :  这种类型非常简单。正如从名称中猜到的那样，它是将接收到的所有消息广播到它知道的 所有队列中。系统中默认有些exchange 类型
## 发送者和接受者
### 发送者代码:
```java
@Component
public class FanoutSender {

    public static String MSG_FROM;

    static {
        MSG_FROM = UUID.randomUUID().toString().replace("-", "");
    }

    @Autowired
    private AmqpTemplate rabbitTemplate;

    public void send(User user) {
        this.rabbitTemplate.convertAndSend(RabbitConfig.FANOUT_EXCHANGE, "", user);
    }

    public void sendMsg(MqMsgDto msg) {
        this.rabbitTemplate.convertAndSend(RabbitConfig.FANOUT_EXCHANGE, "", msg);
    }
}
```
### 接收者代码:
```java
@Component
public class FanoutReceiver {
    // queues是指要监听的队列的名字
    @RabbitListener(queues = RabbitConfig.FANOUT_QUEUE1)
    public void receiveTopic1(MqMsgDto msg) {
        System.out.println("【项目A,receiveFanout监听到消息】" + msg);
        if (!MSG_FROM.equals(msg.getFrom())) {
            //mq转发--RECEIVE_MESSAGE--
            System.out.println("项目A,mq转发--RECEIVE_MESSAGE--" + msg);
        }
    }
}
```
- 请注意 **MSG_FROM** 该静态变量是一个标识,标识消息从哪个应用发出,在接收消息时,判断消息是否为"自己"发出的,是:不转发,否:转发

## 具体业务类
### api : 
```java
@RestController
public class MsgController {

    @Autowired
    private SendMsgEvent sender;

    @PostMapping(value = "/msg/send")
    private void snedMsg(@RequestBody MsgDto msg) {
        MqMsgDto mqMsg = new MqMsgDto(MSG_FROM, msg);
        sender.setMsg(mqMsg);
    }
}
```
### 消息数据类
```java
public class MsgDto implements Serializable {

    private String msg;

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public MsgDto(String msg) {
        this.msg = msg;
    }

    public MsgDto() {
    }

    @Override
    public String toString() {
        return "MsgDto{" +
                "msg='" + msg + '\'' +
                '}';
    }
}
```
### MQ数据传输类
```java
public class MqMsgDto implements Serializable {

    private String from;
    private MsgDto mag;

    public String getFrom() {
        return from;
    }

    public void setFrom(String from) {
        this.from = from;
    }

    public MsgDto getMag() {
        return mag;
    }

    public void setMag(MsgDto mag) {
        this.mag = mag;
    }

    public MqMsgDto(String from, com.spt.chat.entity.MsgDto mag) {
        this.from = from;
        this.mag = mag;
    }

    @Override
    public String toString() {
        return "MqMsgDto{" +
                "from='" + from + '\'' +
                ", mag=" + mag +
                '}';
    }
}
```
业务类:
```java
@Component
public class SendMsgEvent {

    @Autowired
    FanoutSender msgSender;

    public void setMsg(MqMsgDto msg){
        //转发RECEIVE_MESSAGE
        System.out.println("A项目转发--RECEIVE_MESSAGE--消息"+msg);
        //将消息发送只mq
        msgSender.sendMsg(msg);
    }
}
```
至此,springBoot整合MQ部分完毕
# 运行&结果
> 同样的方式,创建子模块:mq_rabbit02,mq_rabbit03  (记得修改yml文件哟~~)
## 运行
### 运行项目
![2022-06-20_run](./rabbitMQ/2022-06-20_run.png)

### 发送消息
![2022-06-20_req_postman](./rabbitMQ/2022-06-20_req_postman.png)

## 结果
### mq_rabbit01
>localhost:8001/msg/send请求到服务mq_rabbit上,mq_rabbit发送消息,监听消息,判断是"自己"发的后,不转发

![2022-06-20_22-01](./rabbitMQ/2022-06-20_22-01.png)
### mq_rabbit02
>mq_rabbit02,监听消息,判断消息不是"自己"发的后,转发

![2022-06-20_22-02](./rabbitMQ/2022-06-20_22-02.png)
### mq_rabbit03
>mq_rabbit03,监听消息,判断消息不是"自己"发的后,转发

![2022-06-20_22-03](./rabbitMQ/2022-06-20_22-03.png)
