高性能的异步通讯组件

## 初始MQ
### 同步调用
同步调用的优势是什么？
- 时效性强，等待到结果后才返回。
同步调用的问题是什么？
- 拓展性差
- 性能下降
- 级联失败问题
![[Pasted image 20240610214203.png]]

### 异步调用
异步调用通常是基于消息通知的方式，包含三个角色：
- 消息发送者：投递消息的人，就是原来的调用者
- 消息接收者：接收和处理消息的人，就是原来的服务提供者
- 消息代理：管理、暂存、转发消息，你可以把它理解成微信服务器

消息代理会一直投送消息给接受者知道成功为止, 所以发送者根本无需关心消息是否接收, 这是代理的活(完全解耦)
![[Pasted image 20240610220016.png]]

支付服务不再同步调用业务关联度低的服务，而是发送消息通知到Broker。
具备下列优势：
- 解除耦合，拓展性强
- 无需等待，性能好
- 故障隔离
- 缓存消息，流量削峰填谷
![[Pasted image 20240610220249.png]]

异调用的优势是什么？
- 耦合度低，拓展性强
- 异步调用，无需等待，性能好
- 故障隔离，下游服务故障不影响上游业务
- 缓存消息，流量削峰填谷
异步调用的问题是什么？
- 不能立即得到调用结果，时效性差
- 不确定下游业务执行是否成功
- 业务安全依赖于Broker的可靠性


### MQ技术选型
MQ （MessageQueue），中文是消息队列，字面来看就是存放消息的队列。也就是异步调用中的Broker。
![[Pasted image 20240610220951.png]]

## 安装和部署
```Shell
docker run \
 -e RABBITMQ_DEFAULT_USER=fancier \
 -e RABBITMQ_DEFAULT_PASS=123456 \
 -v mq-plugins:/plugins \
 --name mq \
 --hostname mq \
 -p 15672:15672 \
 -p 5672:5672 \
 --network hm-net\
 -d \
 rabbitmq:3.8-management
```

## 基本价绍

RabbitMQ的整体架构及核心概念：
- virtual-host：虚拟主机，起到数据隔离的作用
- publisher：消息发送者
- consumer：消息的消费者
- queue：队列，存储消息
- exchange：交换机，负责路由消息
![[Pasted image 20240610223538.png]]

## 快速入门
需求：在RabbitMQ的控制台完成下列操作:
- 新建队列hello.queue1和hello.queue2
- 向默认的amp.fanout交换机发送一条消息
- 查看消息是否到达hello.queue1和hello.queue2
- 总结规律

在交换机和队列之间建立关系

消息发送的注意事项有哪些？

- 交换机只能路由消息，无法存储消息

- 交换机只会路由消息给与其绑定的队列，因此队列必须与交换机绑定

## 数据隔离
需求：在RabbitMQ的控制台完成下列操作:
- 新建一个用户hmall
- 为hmall用户创建一个virtual host
- 测试不同virtual host之间的数据隔离现象

## Java 客户端

### 快速入门
![[Pasted image 20240610230656.png]]

AMQP
Advanced Message Queuing Protocol，是用于在应用程序之间传递业务消息的开放标准。该协议与语言和**平台无关**，更符合微服务中独立性的要求。

需求如下：
- 利用控制台创建队列simple.queue
- 在publisher服务中，利用SpringAMQP直接向simple.queue发送消息
- 在consumer服务中，利用SpringAMQP编写消费者，监听simple.queue队列

#### 1. 引入 依赖
在父工程中引入spring-amqp依赖，这样publisher和consumer服务都可以使用：
```xml
<!--AMQP依赖，包含RabbitMQ-->  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-amqp</artifactId>  
</dependency>
```

#### 2. 配置RabbitMQ服务端信息
```yml
spring:  
  rabbitmq:  
    host: 192.168.200.134 # 你的虚拟机IP  
    port: 5672 # 端口  
    virtual-host: /hmall # 虚拟主机  
    username: hmall # 用户名  
    password: 123456 # 密码
```




#### 3. 发送消息
```java
@Autowired  
private RabbitTemplate rabbitTemplate;  
  
@Test  
public void testSimpleQueue() throws Exception {  
    // 1. 队列名  
    String queueName = "simple.queue";  
    // 2. 消息  
    String message = "hello,world";  
    // 3. 发送消息  
    rabbitTemplate.convertAndSend(queueName, message);  
}
```
#### 4. 接收消息
SpringAMQP提供声明式的消息监听，我们只需要通过注解在方法上声明要监听的队列名称，将来SpringAMQP就会把消息传递给当前方法：
```java
@Slf4j  
@Component  
public class SpringRabbitListener {
	@RabbitListener(queues = "simple.queue")
	public void listenSimpleQueueMessage(String msg) throws InterruptedException {       
		log.info("spring 消费者接收到消息：【" + msg + "】");   
    }
}
```
### WorkQueue
Work queues，任务模型。简单来说就是让多个消费者绑定到一个队列，共同消费队列中的消息。
![[Pasted image 20240611104311.png]]

##### 模拟WorkQueue，实现一个队列绑定多个消费者
基本思路如下：
1. 在RabbitMQ的控制台创建一个队列，名为work.queue
2. 在publisher服务中定义测试方法，发送50条消息到work.queue
3. 在consumer服务中定义两个消息监听者，都监听work.queue队列
4. 消费者1每秒处理40条消息，消费者2每秒处理5条消息

consumer
```java
@Test  
public void testSimpleQueue() throws Exception {  
    // 1. 队列名  
    String queueName = "simple.queue";  
    // 2. 消息  
    String message = "hello,world";  
    // 3. 发送消息  
    rabbitTemplate.convertAndSend(queueName, message);  
}  
  
@Test  
public void testWorkQueue() throws Exception {  
    // 1. 队列名  
    String queueName = "work.queue";  
    // 2. 消息  
    String message = "message";  
    // 3. 发送消息  
    for (int i = 0; i < 50; i++) {  
        rabbitTemplate.convertAndSend(queueName, message + i);  
    }  
}
```

publish
```java
@Test  
public void testWorkQueue() throws Exception {  
    // 1. 队列名  
    String queueName = "work.queue";  
    // 2. 消息  
    String message = "message";  
    // 3. 发送消息  
    for (int i = 0; i < 50; i++) {  
        rabbitTemplate.convertAndSend(queueName, message + i);  
    }  
}
```

同一个消息只能被一个消费者处理, 如果有多个消息会平均分配给每个消费者

能者多劳 消费完了 才能再拿消息
默认情况下，RabbitMQ的会将消息依次轮询投递给绑定在队列上的每一个消费者。但这并没有考虑到消费者是否已经处理完消息，可能出现消息堆积。
因此我们需要修改application.yml，设置preFetch值为1，确保同一时刻最多投递给消费者1条消息：
```YAML
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1 # 每次只能获取一条消息，处理完成才能获取下一个消息
```

#### 总结
Work模型的使用：
- 多个消费者绑定到一个队列，可以加快消息处理速度
- 同一条消息只会被一个消费者处理
- 通过设置prefetch来控制消费者预取的消息数量，处理完一条再处理下一条，实现能者多劳


### Fanout 交换机
交换机的作用主要是接收发送者发送的消息，并将消息路由到与其绑定的队列。
常见交换机的类型有以下三种：
- Fanout：广播
- Direct：定向
- Topic：话题
![[Pasted image 20240611113741.png]]

Fanout Exchange 会将接收到的消息路由到每一个跟其绑定的queue，所以也叫广播模式

![[Pasted image 20240611113846.png]]

#### 利用SpringAMQP演示FanoutExchange的使用

实现思路如下：
1. 在RabbitMQ控制台中，声明队列fanout.queue1和fanout.queue2
2. 在RabbitMQ控制台中，声明交换机hmall.fanout，将两个队列与其绑定
3. 在consumer服务中，编写两个消费者方法，分别监听fanout.queue1和fanout.queue2
4. 在publisher中编写测试方法，向hmall.fanout发送消息

交换机的作用是什么？
- 接收publisher发送的消息
- 将消息按照规则路由到与之绑定的队列
- FanoutExchange的会将消息路由到每个绑定的队列
发送消息到交换机的API是怎样的？
```Java
@Test
public void testSendDirectExchange() {
    // 交换机名称
    String exchangeName = "hmall.direct";
    // 消息
    String message = "红色警报！日本乱排核废水，导致海洋生物变异，惊现哥斯拉！";
    // 发送消息
    rabbitTemplate.convertAndSend(exchangeName, "red", message);
}
```

### Direct交换机
在Fanout模式中，一条消息，会被所有订阅的队列都消费。但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的Exchange。
![[Pasted image 20240611115630.png]]
在Direct模型下：

- 队列与交换机的绑定，不能是任意绑定了，而是要指定一个`RoutingKey`（路由key）
- 消息的发送方在 向 Exchange发送消息时，也必须指定消息的 `RoutingKey`。
- Exchange不再把消息交给每一个绑定的队列，而是根据消息的`Routing Key`进行判断，只有队列的`Routingkey`与消息的 `Routing key`完全一致，才会接收到消息

#### 利用SpringAMQP演示DirectExchange的使用
1. 声明一个名为`hmall.direct`的交换机
2. 声明队列`direct.queue1`，绑定`hmall.direct`，`bindingKey`为`blud`和`red`
3. 声明队列`direct.queue2`，绑定`hmall.direct`，`bindingKey`为`yellow`和`red`
4. 在`consumer`服务中，编写两个消费者方法，分别监听direct.queue1和direct.queue2
5. 在publisher中编写测试方法，向`hmall.direct`发送消息

```java
// 3. 发送消息  
rabbitTemplate.convertAndSend(exchangeName, "yellow", message);
```

```bash
消费者2接收到消息 : 红色消息 : message  时间:2024-06-11T12:02:22.668
消费者1接收到消息 : 红色消息 : message  时间:2024-06-11T12:02:22.669
消费者1接收到消息 : 蓝色消息 : message  时间:2024-06-11T12:02:49.530
消费者2接收到消息 : 黄色消息 : message  时间:2024-06-11T12:03:18.505
```


### Topic交换机
#### 说明
`Topic`类型的`Exchange`与`Direct`相比，都是可以根据`RoutingKey`把消息路由到不同的队列。
只不过`Topic`类型`Exchange`可以让队列在绑定`BindingKey` 的时候使用通配符！

`BindingKey` 一般都是有一个或多个单词组成，多个单词之间以`.`分割，例如： `item.insert`

通配符规则：
- `#`：匹配一个或多个词
- `*`：匹配不多不少恰好1个词

![[Pasted image 20240611214307.png]]

#### 消息接收
```Java
@RabbitListener(queues = "topic.queue1")
public void listenTopicQueue1(String msg){
    System.out.println("消费者1接收到topic.queue1的消息：【" + msg + "】");
}

@RabbitListener(queues = "topic.queue2")
public void listenTopicQueue2(String msg){
    System.out.println("消费者2接收到topic.queue2的消息：【" + msg + "】");
}
```


### 声明队列交换机
在之前我们都是基于RabbitMQ控制台来创建队列、交换机。但是在实际开发时，队列和交换机是程序员定义的，将来项目上线，又要交给运维去创建。那么程序员就需要把程序中运行的所有队列和交换机都写下来，交给运维。在这个过程中是很容易出现错误的。
因此推荐的做法是由程序启动时检查队列和交换机是否存在，如果不存在自动创建。

direct 示例
```Java
package com.itheima.consumer.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DirectConfig {

    /**
     * 声明交换机
     * @return Direct类型交换机
     */
    @Bean
    public DirectExchange directExchange(){
        return ExchangeBuilder.directExchange("hmall.direct").build();
    }

    /**
     * 第1个队列
     */
    @Bean
    public Queue directQueue1(){
        return new Queue("direct.queue1");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1WithRed(Queue directQueue1, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue1).to(directExchange).with("red");
    }
    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1WithBlue(Queue directQueue1, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue1).to(directExchange).with("blue");
    }

    /**
     * 第2个队列
     */
    @Bean
    public Queue directQueue2(){
        return new Queue("direct.queue2");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2WithRed(Queue directQueue2, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue2).to(directExchange).with("red");
    }
    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2WithYellow(Queue directQueue2, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue2).to(directExchange).with("yellow");
    }
}
```

#### 基于注解的方式
```Java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue1"),
    exchange = @Exchange(name = "hmall.topic", type = ExchangeTypes.TOPIC),
    key = "china.#"
))
public void listenTopicQueue1(String msg){
    System.out.println("消费者1接收到topic.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue2"),
    exchange = @Exchange(name = "hmall.topic", type = ExchangeTypes.TOPIC),
    key = "#.news"
))
public void listenTopicQueue2(String msg){
    System.out.println("消费者2接收到topic.queue2的消息：【" + msg + "】");
}
```

### 消息转换器
如果直接发 实体对象 发送消息默认 使用jdk 的转换器
```Java
package com.itheima.consumer.config;

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MessageConfig {

    @Bean
    public Queue objectQueue() {
        return new Queue("object.queue");
    }
}
```

会出现这种情况
![[Pasted image 20240611215539.png]]

非常不友好
#### 配置JSON转换器
显然，JDK序列化方式并不合适。我们希望消息体的体积更小、可读性更高，因此可以使用JSON方式来做序列化和反序列化。

在`publisher`和`consumer`两个服务中都引入依赖：

```XML
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.10</version>
</dependency>
```

注意，如果项目中引入了`spring-boot-starter-web`依赖，则无需再次引入`Jackson`依赖。

```Java
@Bean
public MessageConverter messageConverter(){
    // 1.定义消息转换器
    Jackson2JsonMessageConverter jackson2JsonMessageConverter = new Jackson2JsonMessageConverter();
    // 2.配置自动创建消息id，用于识别不同消息，也可以在业务中基于ID判断是否是重复消息
    jackson2JsonMessageConverter.setCreateMessageIds(true);
    return jackson2JsonMessageConverter;
}
```

## 业务改造

### 配置MQ

不管是生产者还是消费者，都需要配置MQ的基本信息。分为两步：

1）添加依赖：

```XML
  <!--消息发送-->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>
```

这样咱就不用feign了, 直接 把消息发给 mq 让mq转发消息给消费者, 
因为要转发给指定的服务器, 就使用 direct 交换器了
### 接收消息
```Java
package com.hmall.trade.listener;

import com.hmall.trade.service.IOrderService;
import lombok.RequiredArgsConstructor;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RequiredArgsConstructor
public class PayStatusListener {

    private final IOrderService orderService;

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "trade.pay.success.queue", durable = "true"),
            exchange = @Exchange(name = "pay.topic"),
            key = "pay.success"
    ))
    public void listenPaySuccess(Long orderId){
        orderService.markOrderPaySuccess(orderId);
    }
}
```

### 发送消息
```Java
private final RabbitTemplate rabbitTemplate;

@Override
@Transactional
public void tryPayOrderByBalance(PayOrderDTO payOrderDTO) {
    // 1.查询支付单
    PayOrder po = getById(payOrderDTO.getId());
    // 2.判断状态
    if(!PayStatus.WAIT_BUYER_PAY.equalsValue(po.getStatus())){
        // 订单不是未支付，状态异常
        throw new BizIllegalException("交易已支付或关闭！");
    }
    // 3.尝试扣减余额
    userClient.deductMoney(payOrderDTO.getPw(), po.getAmount());
    // 4.修改支付单状态
    boolean success = markPayOrderSuccess(payOrderDTO.getId(), LocalDateTime.now());
    if (!success) {
        throw new BizIllegalException("交易已支付或关闭！");
    }
    // 5.修改订单状态
    // tradeClient.markOrderPaySuccess(po.getBizOrderNo());
    try {
        rabbitTemplate.convertAndSend("pay.direct", "pay.success", po.getBizOrderNo());
    } catch (Exception e) {
        log.error("支付成功的消息发送失败，支付单id：{}， 交易单id：{}", po.getId(), po.getBizOrderNo(), e);
    }
}
```

# 高级

## 消息可靠性问题
因此，这里我们必须尽可能确保MQ消息的可靠性，即：消息应该至少被消费者处理1次
那么问题来了：
- **我们该如何确保****MQ****消息的可靠性**？
- **如果真的发送失败，有没有其它的兜底方案？**

### 发送者的可靠性
首先，我们一起分析一下消息丢失的可能性有哪些。
消息从发送者发送消息，到消费者处理消息，需要经过的流程是这样的：
暂时无法在飞书文档外展示此内容
消息从生产者到消费者的每一步都可能导致消息丢失：
- 发送消息时丢失：
    - 生产者发送消息时连接MQ失败
    - 生产者发送消息到达MQ后未找到`Exchange`
    - 生产者发送消息到达MQ的`Exchange`后，未找到合适的`Queue`
    - 消息到达MQ后，处理消息的进程发生异常
- MQ导致消息丢失：
    - 消息到达MQ，保存到队列后，尚未消费就突然宕机
- 消费者处理消息时：
    - 消息接收后尚未处理突然宕机
    - 消息接收后处理过程中抛出异常
综上，我们要解决消息丢失问题，保证MQ的可靠性，就必须从3个方面入手：
- 确保生产者一定把消息发送到MQ
- 确保MQ不会将消息弄丢   
- 确保消费者一定要处理消息
#### 发送者重连
首先第一种情况，就是生产者发送消息时，出现了网络故障，导致与MQ的连接中断。

为了解决这个问题，SpringAMQP提供的消息发送时的重试机制。即：当`RabbitTemplate`与MQ连接超时后，多次重试。
修改`publisher`模块的`application.yaml`文件，添加下面的内容：

```YAML
spring:
  rabbitmq:
    connection-timeout: 1s # 设置MQ的连接超时时间
    template:
      retry:
        enabled: true # 开启超时重试机制
        initial-interval: 1000ms # 失败后的初始等待时间
        multiplier: 1 # 失败后下次的等待时长倍数，下次等待时长 = initial-interval * multiplier
        max-attempts: 3 # 最大重试次数
```
#### 发送者确认
SpringAMQP提供了Publisher Confirm和Publisher Return两种确认机制。开启确机制认后，当发送者发送消息给MQ后，MQ会返回确认结果给发送者。返回的结果有以下几种情况：
- 消息投递到了MQ，但是路由失败(没有配队列)。此时会通过PublisherReturn返回路由异常原因，然后返回ACK，告知投递成功
- 临时消息投递到了MQ，并且入队成功，返回ACK，告知投递成功
- 持久消息投递到了MQ，并且入队完成持久化，返回ACK ，告知投递成功
 - 其它情况都会返回NACK，告知投递失败
![[Pasted image 20240612085037.png]]

##### SpringAMQP实现发送者确认
```YAML
spring:
  rabbitmq:
    publisher-confirm-type: correlated # 开启publisher confirm机制，并设置confirm类型
    publisher-returns: true # 开启publisher return机制
```
这里`publisher-confirm-type`有三种模式可选：
- `none`：关闭confirm机制
- `simple`：同步阻塞等待MQ的回执
- `correlated`：MQ异步回调返回回执


###### ReturnCallBack
```Java
@Slf4j
@AllArgsConstructor
@Configuration
public class MqConfig {
    private final RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init(){
        rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
            @Override
            public void returnedMessage(ReturnedMessage returned) {
                log.error("触发return callback,");
                log.debug("exchange: {}", returned.getExchange());
                log.debug("routingKey: {}", returned.getRoutingKey());
                log.debug("message: {}", returned.getMessage());
                log.debug("replyCode: {}", returned.getReplyCode());
                log.debug("replyText: {}", returned.getReplyText());
            }
        });
    }
}
```


###### ConfirmCallBack
```Java
@Test
void testPublisherConfirm() {
    // 1.创建CorrelationData
    CorrelationData cd = new CorrelationData();
    // 2.给Future添加ConfirmCallback
    cd.getFuture().addCallback(new ListenableFutureCallback<CorrelationData.Confirm>() {
        @Override
        public void onFailure(Throwable ex) {
            // 2.1.Future发生异常时的处理逻辑，基本不会触发
            log.error("send message fail", ex);
        }
        @Override
        public void onSuccess(CorrelationData.Confirm result) {
            // 2.2.Future接收到回执的处理逻辑，参数中的result就是回执内容
            if(result.isAck()){ // result.isAck()，boolean类型，true代表ack回执，false 代表 nack回执
                log.debug("发送消息成功，收到 ack!");
            }else{ // result.getReason()，String类型，返回nack时的异常描述
                log.error("发送消息失败，收到 nack, reason : {}", result.getReason());
            }
        }
    });
    // 3.发送消息
    rabbitTemplate.convertAndSend("hmall.direct", "q", "hello", cd);
}
```
### MQ的可靠性
#### 数据持久化
- 交换机持久化
- 队列持久化
- 消息持久化
不设置持久化, 服务器关了 数据

SpringAMQP 发送的消息默认是持久化的

怎么构建非持久化 消息
![[Pasted image 20240612092008.png]]

#### Lazy Queue
因为每发一条消息都要往磁盘存, 延长的时间将导致并发能力下降
\
从RabbitMQ的3.6.0版本开始，就增加了Lazy Queue的概念，也就是惰性队列。
惰性队列的特征如下：
- 接收到消息后直接存入磁盘，不再存储到内存
- 消费者要消费消息时才会从磁盘中读取并加载到内存（可以提前缓存部分消息到内存，最多2048条）
在3.12版本后，所有队列都是Lazy Queue模式，无法更改。

要设置一个队列为惰性队列，只需要在声明队列时，指定x-queue-mode属性为lazy即可：
![[Pasted image 20240612092943.png]]

```Java
@Bean
public Queue lazyQueue(){
    return QueueBuilder
            .durable("lazy.queue")
            .lazy() // 开启Lazy模式
            .build();
}
```

```Java
@RabbitListener(queuesToDeclare = @Queue(
        name = "lazy.queue",
        durable = "true",
        arguments = @Argument(name = "x-queue-mode", value = "lazy")
))
public void listenLazyQueue(String msg){
    log.info("接收到 lazy.queue的消息：{}", msg);
}
```

### 消费者的可靠性


### 延迟消息













