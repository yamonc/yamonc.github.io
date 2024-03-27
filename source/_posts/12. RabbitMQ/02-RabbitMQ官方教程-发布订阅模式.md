---
title: RabbitMQ官方教程-Publish/Subscribe
date: 2023-08-30 15:57:19
tags: [RabbitMQ]
categories: [消息队列]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301556832.jpeg
comment: 是
keywords: 
password: 
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# RabbitMQ官方教程-Publish/Subscribe

![a scuba diver swims through an underwater cave](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301556832.jpeg)

在之前的教程汇总，我们创建了一个工作队列。这种假设是基于每个任务只会传递给一个worker。在这一部分中，我们将做一些完全不同的事情——我们将向多个消费者传递消息。这种模式称为“发布/订阅”。

为了说明该模式，我们将构建一个简单的日志系统。它将由两个程序组成——第一个程序将发出日志消息，第二个程序将接收并打印它们。

在我们的日志系统中，接收程序的每个正在运行的副本都会收到消息。这样我们就能运行一个接收器并将日志定向到磁盘；同时我们能运行另外一个接收器并在屏幕上查看日志。

本质上，发布的日志消息将广播给所有的接收者。

## Exchanges

在本教程的前面部分中，我们向消息队列发送消息和从消息队列中接收消息。

现在是时候介绍RabbitMQ中完整的消息传递模型了。

让我们快速回顾一下前面教程中介绍的内容：

- 生产者是发送消息的用户应用程序。

- 队列是存储消息的缓冲区。

- 消费者是接收消息的用户应用程序。

Rabbit MQ 消息传递模型的核心思想是生产者从不直接向队列发送任何消息。实际上，生产者通常根本不知道消息是否会被传递到任何队列。

相反，生产者只能将消息发送到exchange。交换是一件非常简单的事情。

一方面，它接收来自生产者的消息，另一方面，它将消息推送到队列。

交换机必须确切地知道如何处理它收到的消息。比如：是否应该将其附加到特定队列？是否应该将其附加到许多队列中？或者应该将其丢弃。其规则由交换类型定义。

![An exchange: The producer can only send messages to an exchange. One side of the exchange receives messages from producers and the other side pushes them to queues.](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301426501.png)

有几种可用的交换类型：direct、topic、headers和fanout。

我们将重点关注最后一个——fanout。让我们创建一个这种类型的交换，并将其称为日志：

```java
channel.exchangeDeclare("logs", "fanout");
```

扇出交换非常简单。正如您可能从名称中猜到的那样，它只是将收到的所有消息广播到它知道的所有队列。这正是我们的记录器所需要的。

> Listing exchanges 
>
> 要列出服务器上的交换机，您可以运行非常有用的rabbitmqctl：
> sudo rabbitmqctl list_exchanges
>
> 在此列表中将有一些 amq.* 交换和默认（未命名）exchanges。这些是默认创建的，但目前您不太可能需要使用它们。
>
> 匿名交换机
>
> 在本教程的前面部分中，我们对交换一无所知，但仍然能够将消息发送到队列。这是可能的，因为我们使用的是默认交换，我们通过空字符串（“”）来标识它。
>
> 回想一下我们之前发布的消息：
>
> ```java
> channel.basicPublish("", "hello", null, message.getBytes());
> ```
>
> 第一个参数是交换机的名称。空字符串表示默认的或无名称的交换：消息将路由到routingKey指定名称的队列（如果存在）。

现在，我们可以发布到我们自己定义的exchange中：

```java
channel.basicPublish( "logs", "", null, message.getBytes());
```

## Temporary queues（临时队列）

您可能还记得之前我们使用具有特定名称的队列（还记得 hello 和 task_queue 吗？）。能够命名队列对我们来说至关重要——我们需要将worker指向同一个队列。当您想要在生产者和消费者之间共享队列时，为队列命名非常重要。

但我们的日志系统并非如此。我们希望了解所有日志消息，而不仅仅是其中的一部分。我们也只对当前流动的消息感兴趣，而不是旧的消息。为了解决这个问题，我们需要两件事。

首先，每当我们连接到 Rabbit 时，我们都需要一个新的空队列。为此，我们可以创建一个具有随机名称的队列，或者更好 - 让服务器为我们选择一个随机队列名称。

其次，一旦我们和消费者consumer断连之后，消息队列应该自动被删除。

在 Java 客户端中，当我们不向queueDeclare()提供任何参数时，我们会创建一个具有生成名称的非持久、独占、自动删除队列：

```java
String queueName = channel.queueDeclare().getQueue();
```

您可以在队列指南中了解有关exclusive标志和其他队列属性的更多信息。

此时queueName包含随机队列名称。例如，它可能看起来像 amq.gen-Jz TY20 BRg KO-Hjm UJj0w Lg。

## Bindings(绑定)

![The exchange sends messages to a queue. The relationship between the exchange and a queue is called a binding.](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301442050.png)

我们已经创建了fanout exchange和队列。现在我们需要告诉交换器将消息发送到我们的队列。交换器和队列之间的关系称为binding：

```java
channel.queueBind(queueName, "logs", "");
```

从现在开始，日志交换会将消息附加到我们的队列中。

> 罗列出bingdings：
>
> ```bash
> rabbitmqctl list_bindings
> ```

## 组装在一起

![Producer -> Queue -> Consuming: deliver a message to multiple consumers.](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301446121.png)

发出日志消息的生产者程序看起来与之前的教程没有太大不同。最重要的变化是我们现在想要将消息发布到我们的logs而不是无名的交换。发送时我们需要提供一个routingKey，但它的值在fanout交换中会被忽略。下面是 Emit Log.java 程序的代码：

```java
package org.example.demo3;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * @Author yamon
 * @Date 2023-08-30 14:49
 * @Description 输出日志
 * @Version 1.0
 */
public class EmitLog {
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

            String message = argv.length < 1 ? "info: Hello World!" :
                    String.join(" ", argv);

            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
```

如您所见，建立连接后我们声明了exchange。此步骤是必要的，因为禁止发布到不存在的exchange。

如果还没有队列绑定到交换器，消息将会丢失，但这对我们来说没关系；如果还没有消费者在监听，我们可以安全地丢弃该消息。

ReceiveLogs.java

```java
package org.example.demo3;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

/**
 * @Author yamon
 * @Date 2023-08-30 14:51
 * @Description
 * @Version 1.0
 */
public class ReceiveLogs {
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
    }
}
```

执行这两个类：建议先消费者，后生产者

![image-20230830145619961](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301456003.png)

![image-20230830145629667](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301456796.png)

在docker desktop上查看：

![image-20230830145657736](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301456797.png)

