---
title: RabbitMQ官方教程-Routing
date: 2023-08-30 15:58:34
tags: [RabbitMQ]
categories: [消息队列]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301558423.jpeg
comment: 是
keywords: 
password: 
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# RabbitMQ官方教程-Routing

![a grassy hill with clouds in the sky](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301558423.jpeg)

在之前的教程中，我们创建了一个简单的日志系统。我们现在可以通过广播的方式将日志信息发送给很多接收者。

在本教程中，我们将向其添加一个功能 - 我们将使其能够仅订阅消息的子集。例如，我们将能够仅将关键错误消息定向到日志文件（以节省磁盘空间），同时仍然能够在控制台上打印所有日志消息。

## 绑定

在前面的示例中，我们已经创建了绑定。

您可能还记得这样的代码：

```java
channel.queueBind(queueName, EXCHANGE_NAME, "");
```

绑定是交换和队列之间的关系。这可以简单地理解为：队列对来自此交换器的消息感兴趣。

绑定可以采用额外的routingKey 参数。为了避免与 basic_publish 参数混淆，我们将其称为binding_key。

这就是我们如何创建带有键的绑定：

```java
channel.queueBind(queueName, EXCHANGE_NAME, "black");
```

绑定key的含义取决于交换类型。我们之前使用的fanout交换完全忽略了它的价值。

## Direct exchange

直连模式

上一篇教程中的日志系统将所有消息广播给所有消费者。

我们希望扩展它以允许根据消息的严重性过滤消息。

例如，我们可能希望一个将日志消息写入磁盘的程序仅接收关键错误，而不是在警告或信息日志消息上浪费磁盘空间。

我们使用的是fanout模式，这并没有给我们带来太大的灵活性——它只能进行无意识的广播。

我们将改用direct exchange。直接交换背后的路由算法很简单：消息进入其binding_key与消息的routing_key完全匹配的队列。

为了解释说明这些，如下图：

![Direct exchange routing](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301522303.png)

在此设置中，我们可以看到直接交换器 X 绑定了两个队列。

第一个队列使用橙色绑定键绑定，第二个队列有两个绑定，一个使用黑色绑定键，另一个使用绿色绑定键。

在这样的设置中，使用路由键橙色发布到交换器的消息将被路由到队列 Q1。路由键为黑色或绿色的消息将发送至 Q2。所有其他消息将被丢弃。

## 多绑定（Multiple Bindings）

![Multiple Bindings](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301527685.png)

使用相同的绑定键绑定多个队列是完全合法的。在我们的示例中，我们可以使用绑定键黑色在 X 和 Q1 之间添加绑定。在这种情况下，直接交换的行为将类似于fanout模式，并将消息广播到所有匹配的队列。带有routingKey黑色的消息将被传递到 Q1 和 Q2。

## 发出日志 Emitting logs

我们将在我们的日志系统中使用这个模型。我们将fanout模式替换成direct模式，通过routing_key来记录严重日志，这样，接收程序将能够选择它想要接收的严重性。

我们首先需要创建一个exchange：

```java
channel.exchangeDeclare(EXCHANGE_NAME, "direct");
```

然后，我们可以发送一个消息了：

```java
channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes());
```

为了简化事情，我们假设“严重性”可以是“信息”、“警告”、“错误”之一。

## Subscribing 订阅

接收消息的工作方式与上一篇教程类似，但有一个例外 - 我们将为我们感兴趣的每个严重性创建一个新的绑定。

```java
String queueName = channel.queueDeclare().getQueue();

for(String severity : argv){
  channel.queueBind(queueName, EXCHANGE_NAME, severity);
}
```

## 汇总到一起

![Final routing: putting it all together.](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301536272.png)

```java
package org.example.demo4;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * @Author yamon
 * @Date 2023-08-30 15:37
 * @Description
 * @Version 1.0
 */
public class EmitLogDirect {
    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.exchangeDeclare(EXCHANGE_NAME, "direct");

            String severity = getSeverity(argv);
            String message = getMessage(argv);

            channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + severity + "':'" + message + "'");
        }
    }
    //..
    private static String getSeverity(String[] argv){
        return "这是一条严重消息";
    }

    private static String getMessage(String[] argv){
        return "得到一条消息";
    }
}
```

```java
package org.example.demo4;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

/**
 * @Author yamon
 * @Date 2023-08-30 15:38
 * @Description
 * @Version 1.0
 */
public class ReceiveLogsDirect {
    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "direct");
        String queueName = channel.queueDeclare().getQueue();

        if (argv.length < 1) {
            System.err.println("Usage: ReceiveLogsDirect [info] [warning] [error]");
            System.exit(1);
        }

        for (String severity : argv) {
            channel.queueBind(queueName, EXCHANGE_NAME, severity);
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
