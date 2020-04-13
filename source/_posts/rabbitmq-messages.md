---
title: RabbitMQ学习-消息类型
date: 2020-04-12 00:34:52
categories: rabbitmq
tags:
---

本文介绍消息类型，共用六种消息类型……

## 基本消息模型

{%asset_img b-1.png%}

上图中：

- P： 生产者，发送消息的程序。   
- C： 消费者，消息的接收者，会一直等待消息的到来，并消费。
- queue： 消息队列，上图红色部分，一般存在内存，也可以持久化到磁盘。存储消息的媒介，生产者投递消息过来，消费者从中获取。

_例子_：

略……

#### 消息确认机制（ACK）

上面的案例中，消息一旦被消费者消费，队列中的消息就会被删除。

那么问题是，RabbitMQ是怎么知道消息已经被接收了的呢？

如果消费者接收到消息后，还没来得及处理就宕机了或者程序抛出异常，消息消费失败，RabbitMQ服务器无从知道，这样消息就丢失了。

因此，RabbitMQ有个ACK机制。当消费之接收到消息后，会向RabbitMQ发送回执ACK，告知消息已经被接收。

ACK有两种方式：

1.自动ACK：消息一旦被接收，消费者自动发送ACK。         
2.手动ACK: 消息被接收后，消费者不自动发送ACK，要程序手动发送ACK。

以上两种方式的选择：

- 如果消息不那么重要，允许消息丢失的情况，那么就选择自动发送ACK方便些。
- 如果消息非常重要，不允许丢失的情况，那么就要在正确的处理好接收的消息后，再手动发送ACK告知RabbitMQ。否则，消息可能丢失。

_示例：_

略……

生产者避免数据丢失：https://www.cnblogs.com/vipstone/p/9350075.html

## work消息模型

工作队列或者竞争消费者模式。

{%asset_img b-2.png%}

该模式与上面的入门模式相比，多了个消费者，两个消费者共同消费同一个队列中的消息，但是一个消息只能被一个消费者消费。

这个消息模式在web应用中特别有用，在复杂http请求中，可以开启多个消费者，相当于开启多个任务处理处理消息，提高响应速度。

>消息分配给消费的方式有两种：
>- 一种是平均的分给每个消费者。哪怕一个消费者很忙，一个很空闲，但是最终，各个消费者消费的消息条数一样。
>- 另外一种方式就是，能者多劳。消费的越快的消费者，消费更多的消息。这种模式下，手动ack的情况下才生效，自动ack不生效。


## Publish/subscribe（交换机类型：Fanout，也称为广播 ）

{%asset_img b-3.png%}

说民：

1.一个生产者，多个消费者。      
2.每个消费者都有自己的一个队列。       
3.生产者没有将消息直接发送给队列，而是发送给exchange(交换机、转发器)。   
4.每个队列都需要绑定到交换机上。       
5.生产者发送的消息，经过交换机到达队列，实现一个消息被多个消费者消费

例子：注册->发邮件、发短信

X（Exchanges）：交换机一方面：接收生产者发送的消息。另一方面：知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。

Exchange类型有以下几种：

Fanout：广播，将消息交给所有绑定到交换机的队列

Direct：定向，把消息交给符合指定routing key 的队列

Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列

Header：header模式与routing不同的地方在于，header模式取消routingkey，使用header中的 key/value（键值对）匹配队列。

Exchange（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失！

_示例：_

略……

_思考：_

1、publish/subscribe与work queues有什么区别。

区别：

1）work queues不用定义交换机，而publish/subscribe需要定义交换机。

2）publish/subscribe的生产方是面向交换机发送消息，work queues的生产方是面向队列发送消息(底层使用默认交换机)。

3）publish/subscribe需要设置队列和交换机的绑定，work queues不需要设置，实际上work queues会将队列绑定到默认的交换机 。

相同点：

所以两者实现的发布/订阅的效果是一样的，多个消费端监听同一个队列不会重复消费消息。

2、实际工作用 publish/subscribe还是work queues。

建议使用 publish/subscribe，发布订阅模式比工作队列模式更强大（也可以做到同一队列竞争），并且发布订阅模式可以指定自己专用的交换机。

## Routing 路由模型（交换机类型：direct）

{%asset_img b-4.png%}

_P_：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。

_X_：Exchange（交换机），接收生产者的消息，然后把消息递交给 与routing key完全匹配的队列

_C1_：消费者，其所在队列指定了需要routing key 为 error 的消息

_C2_：消费者，其所在队列指定了需要routing key 为 info、error、warning 的消息


例子：

发送者sender：

    public class Send {
        private final static String EXCHANGE_NAME = "test_direct_exchange";
     
        public static void main(String[] argv) throws Exception {
            // 获取到连接
            Connection connection = ConnectionUtil.getConnection();
            // 获取通道
            Channel channel = connection.createChannel();
            // 声明exchange，指定类型为direct
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
            // 消息内容，
            String message = "注册成功！请短信回复[T]退订";
            // 发送消息，并且指定routing key 为：sms，只有短信服务能接收到消息
            channel.basicPublish(EXCHANGE_NAME, "sms", null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
     
            channel.close();
            connection.close();
        }
    }
    
消费者1：
  
    public class Recv {
        private final static String QUEUE_NAME = "direct_exchange_queue_sms";//短信队列
        private final static String EXCHANGE_NAME = "test_direct_exchange";
     
        public static void main(String[] argv) throws Exception {
            // 获取到连接
            Connection connection = ConnectionUtil.getConnection();
            // 获取通道
            Channel channel = connection.createChannel();
            // 声明队列
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            
            // 绑定队列到交换机，同时指定需要订阅的routing key。可以指定多个
            channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "sms");//指定接收发送方指定routing key为sms的消息
            //channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "email");
     
            // 定义队列的消费者
            DefaultConsumer consumer = new DefaultConsumer(channel) {
                // 获取消息，并且处理，这个方法类似事件监听，如果有消息的时候，会被自动调用
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
                                           byte[] body) throws IOException {
                    // body 即消息体
                    String msg = new String(body);
                    System.out.println(" [短信服务] received : " + msg + "!");
                }
            };
            // 监听队列，自动ACK
            channel.basicConsume(QUEUE_NAME, true, consumer);
        }
    }    
    
消费者2：
  
    public class Recv2 {
        private final static String QUEUE_NAME = "direct_exchange_queue_email";//邮件队列
        private final static String EXCHANGE_NAME = "test_direct_exchange";
     
        public static void main(String[] argv) throws Exception {
            // 获取到连接
            Connection connection = ConnectionUtil.getConnection();
            // 获取通道
            Channel channel = connection.createChannel();
            // 声明队列
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            
            // 绑定队列到交换机，同时指定需要订阅的routing key。可以指定多个
            channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "email");//指定接收发送方指定routing key为email的消息
     
            // 定义队列的消费者
            DefaultConsumer consumer = new DefaultConsumer(channel) {
                // 获取消息，并且处理，这个方法类似事件监听，如果有消息的时候，会被自动调用
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
                                           byte[] body) throws IOException {
                    // body 即消息体
                    String msg = new String(body);
                    System.out.println(" [邮件服务] received : " + msg + "!");
                }
            };
            // 监听队列，自动ACK
            channel.basicConsume(QUEUE_NAME, true, consumer);
        }
    }  

我们发送sms的RoutingKey，发现结果：只有指定短信的消费者1收到消息了。

## Topics 通配符模式（交换机类型：topics）

 每个消费者监听自己的队列，并且设置带统配符的routingkey,生产者将消息发给broker，由交换机根据routingkey来转发消息到指定的队列。

Routingkey一般都是有一个或者多个单词组成，多个单词之间以“.”分割，例如：inform.sms

通配符规则：

`#`：匹配一个或多个词

`*`：匹配不多不少恰好1个词

举例：

audit.#：能够匹配audit.irs.corporate 或者 audit.irs

audit.*：只能匹配audit.irs

从示意图可知，我们将发送所有描述动物的消息。消息将使用由三个字（两个点）组成的Routing key发送。路由关键字中的第一个单词将描述速度，第二个颜色和第三个种类：“<speed>.<color>.<species>”。

我们创建了三个绑定：Q1绑定了“*.orange.*”，Q2绑定了“.*.*.rabbit”和“lazy.＃”。

Q1匹配所有的橙色动物。

Q2匹配关于兔子以及懒惰动物的消息。

 下面做个小练习，假如生产者发送如下消息，会进入哪个队列：

quick.orange.rabbit       Q1 Q2   routingKey="quick.orange.rabbit"的消息会同时路由到Q1与Q2

lazy.orange.elephant    Q1 Q2

quick.orange.fox           Q1

lazy.pink.rabbit              Q2  (值得注意的是，虽然这个routingKey与Q2的两个bindingKey都匹配，但是只会投递Q2一次)

quick.brown.fox            不匹配任意队列，被丢弃

quick.orange.male.rabbit   不匹配任意队列，被丢弃

orange         不匹配任意队列，被丢弃

下面我们以指定Routing key="quick.orange.rabbit"为例，验证上面的答案

生产者：

    public class Send {
        private final static String EXCHANGE_NAME = "test_topic_exchange";
     
        public static void main(String[] argv) throws Exception {
            // 获取到连接
            Connection connection = ConnectionUtil.getConnection();
            // 获取通道
            Channel channel = connection.createChannel();
            // 声明exchange，指定类型为topic
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
            // 消息内容
            String message = "这是一只行动迅速的橙色的兔子";
            // 发送消息，并且指定routing key为：quick.orange.rabbit
            channel.basicPublish(EXCHANGE_NAME, "quick.orange.rabbit", null, message.getBytes());
            System.out.println(" [动物描述：] Sent '" + message + "'");
     
            channel.close();
            connection.close();
        }
    }

消费者1：

    public class Recv {
        private final static String QUEUE_NAME = "topic_exchange_queue_Q1";
        private final static String EXCHANGE_NAME = "test_topic_exchange";
     
        public static void main(String[] argv) throws Exception {
            // 获取到连接
            Connection connection = ConnectionUtil.getConnection();
            // 获取通道
            Channel channel = connection.createChannel();
            // 声明队列
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            
            // 绑定队列到交换机，同时指定需要订阅的routing key。订阅所有的橙色动物
            channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "*.orange.*");
     
            // 定义队列的消费者
            DefaultConsumer consumer = new DefaultConsumer(channel) {
                // 获取消息，并且处理，这个方法类似事件监听，如果有消息的时候，会被自动调用
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
                                           byte[] body) throws IOException {
                    // body 即消息体
                    String msg = new String(body);
                    System.out.println(" [消费者1] received : " + msg + "!");
                }
            };
            // 监听队列，自动ACK
            channel.basicConsume(QUEUE_NAME, true, consumer);
        }
    }
    
消费者2：

    public class Recv2 {
        private final static String QUEUE_NAME = "topic_exchange_queue_Q2";
        private final static String EXCHANGE_NAME = "test_topic_exchange";
     
        public static void main(String[] argv) throws Exception {
            // 获取到连接
            Connection connection = ConnectionUtil.getConnection();
            // 获取通道
            Channel channel = connection.createChannel();
            // 声明队列
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            
            // 绑定队列到交换机，同时指定需要订阅的routing key。订阅关于兔子以及懒惰动物的消息
            channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "*.*.rabbit");
            channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "lazy.＃");
     
            // 定义队列的消费者
            DefaultConsumer consumer = new DefaultConsumer(channel) {
                // 获取消息，并且处理，这个方法类似事件监听，如果有消息的时候，会被自动调用
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
                                           byte[] body) throws IOException {
                    // body 即消息体
                    String msg = new String(body);
                    System.out.println(" [消费者2] received : " + msg + "!");
                }
            };
            // 监听队列，自动ACK
            channel.basicConsume(QUEUE_NAME, true, consumer);
        }
    }
    
结果C1、C2是都接收到消息了。

## RPC

{%asset_img b-6.png%}

基本概念：

Callback queue 回调队列，客户端向服务器发送请求，服务器端处理请求后，将其处理结果保存在一个存储体中。而客户端为了获得处理结果，那么客户在向服务器发送请求时，同时发送一个回调队列地址reply_to。

Correlation id 关联标识，客户端可能会发送多个请求给服务器，当服务器处理完后，客户端无法辨别在回调队列中的响应具体和那个请求时对应的。为了处理这种情况，客户端在发送每个请求时，同时会附带一个独有correlation_id属性，这样客户端在回调队列中根据correlation_id字段的值就可以分辨此响应属于哪个请求。

流程说明：

当客户端启动的时候，它创建一个匿名独享的回调队列。
在 RPC 请求中，客户端发送带有两个属性的消息：一个是设置回调队列的 reply_to 属性，另一个是设置唯一值的 correlation_id 属性。
将请求发送到一个 rpc_queue 队列中。
服务器等待请求发送到这个队列中来。当请求出现的时候，它执行他的工作并且将带有执行结果的消息发送给 reply_to 字段指定的队列。
客户端等待回调队列里的数据。当有消息出现的时候，它会检查 correlation_id 属性。如果此属性的值与请求匹配，将它返回给应用

    




