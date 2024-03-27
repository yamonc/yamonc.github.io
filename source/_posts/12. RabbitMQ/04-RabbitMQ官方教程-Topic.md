---
title: 04-RabbitMQ官方教程-topic
date: 2023-08-31 15:56:07
tags: [RabbitMQ]
categories: [消息队列]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308311555612.jpeg
comment: 是
keywords: 
password: 
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# 04-RabbitMQ官方教程-topic

![a large body of water surrounded by mountains](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308311555612.jpeg)

在上一个教程中，我们改进了日志系统。

我们不使用fanout的交换器，而是使用了直接广播，而是使用了一个直接的交换器，并获得了有选择性接收日志的可能性。

虽然，使用direct交换器可以提升我们的系统，但是他仍然有局限性——不能基于多个标准进行路由。

在我们的日志系统中，我们可能不仅希望根据严重性来订阅日志，还希望根据发出日志的源头来订阅日志。

我们可能从syslog工具中了解这个概念，该工具根据严重性（info、warn、crit）和功能（auth、cron、kern）路由日志。

这种行为会给我们很多的灵活性，我们可能只想听取来自cron的关键错误，也希望听取来自kern的所有日志。

为了提升，我们在日志系统中可以学习并添加topic的交换器来完成该功能。

## Topic exchange

发送到topic exchange的消息不能有任意的routing_key - 它必须是一个由点分隔的单词列表。

这些单词可以是任何内容，但通常它们指定与消息相关的一些功能。

一些有效的路由键示例：“stock.usd.nyse”、“nyse.vmw”、“quick.orange.rabbit”。

路由秘钥中可以有任意多个单词，最多 255 个字节。

绑定key也必须采用相同的形式。topic exchang背后的逻辑与direct exchange类似，使用特定路由key发送的消息将被传送到匹配的绑定建绑定的所有队列，然而，绑定建有两种重要的特殊情况：

- *可以代替一个单词
- #可以0或多个单词

举个例子：

![Topic Exchange illustration, which is all explained in the following text.](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308311355816.png)

在此示例中，我们将发送所有描述动物的消息。消息由三个单词（两个点）组成的routing key发送。routing key中第一个单词描述速度，第二个单词描述颜色，第三个单词描述物种。<speed>.<colour>.<species>"

我们创建了三个绑定：

Q1使用绑定键”\*.orange.*“ 

Q2使用绑定键“\*.*rabbit"和"lazy.#"绑定

这些绑定描述如下：

Q1对所有颜色为orange的动物感兴趣。

Q2希望接收到所有rabbits的消息，以及关于lazy动物的一切。

比如：

routing key设置为“quick.orange.rabbit”的消息将被传递到两个队列。

消息“lazy.orange.elephant”也将发送给他们两人。

另一方面，“quick.orange.fox”只会进入第一个队列，而“lazy.brown.fox”只会进入第二个队列。

“lazy.pink.rabbit”只会被传递到第二个队列一次，即使它匹配两个绑定。

“quick.brown.fox”与任何绑定都不匹配，因此它将被丢弃。

> Topic exchange
>
> Topic exchange很强大，可以和其他交换器一起进行。
>
> 当一个队列绑定 # 表示，它介绍所有的消息。这时候就和fanout模式的一致。
>
> 当不使用的字符*和#的时候，就和direct模式一致。

## 整合

我们在日志系统中使用topic exchange。首先假设日志的路由键有两个：facility和severity

code for EmitLogTopic.java

```java
private static final String EXCHANGE_NAME = "topic_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "topic");
    String queueName = channel.queueDeclare().getQueue();

    if (argv.length < 1) {
        System.err.println("Usage: ReceiveLogsTopic [binding_key]...");
        System.exit(1);
    }

    for (String bindingKey : argv) {
        channel.queueBind(queueName, EXCHANGE_NAME, bindingKey);
    }

    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    DeliverCallback deliverCallback = (consumerTag, delivery) -> {
        String message = new String(delivery.getBody(), "UTF-8");
        System.out.println(" [x] Received '" +
            delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
    };
    channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
  }
```

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class ReceiveLogsTopic {

  private static final String EXCHANGE_NAME = "topic_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "topic");
    String queueName = channel.queueDeclare().getQueue();

    if (argv.length < 1) {
        System.err.println("Usage: ReceiveLogsTopic [binding_key]...");
        System.exit(1);
    }

    for (String bindingKey : argv) {
        channel.queueBind(queueName, EXCHANGE_NAME, bindingKey);
    }

    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    DeliverCallback deliverCallback = (consumerTag, delivery) -> {
        String message = new String(delivery.getBody(), "UTF-8");
        System.out.println(" [x] Received '" +
            delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
    };
    channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
  }
}
```
