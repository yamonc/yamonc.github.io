---
title: RabbitMQ官方教程-hello world
date: 2023-08-30 15:55:29
tags: [RabbitMQ]
categories: [消息队列]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301554697.jpeg
comment: 是
keywords: 
password: 
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# RabbitMQ官方教程-hello world

![a man standing on top of a cliff overlooking a lake](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301554697.jpeg)

## 先决条件

本教程假设RabbitMQ已经安装并在标准端口（5672）上的本机主机运行。如果使用不同的主机、端口或凭证，则需要调整连接设置。

## 引入

rabbit MQ 是一个消息代理：它接受并转发消息。

您可以将其视为邮局：当您将要投递的邮件放入邮箱时，您可以确定邮递员最终会将邮件递送给收件人。

在这个类比中，RabbitMQ 是一个邮箱、一个邮局和一个信递员。

Rabbit MQ 和邮局之间的主要区别在于，它不处理纸张，而是接受、存储和转发二进制数据块（消息）。

Rabbitmq 和一般的消息传递使用一些术语：

- 生产无非就是发送。发送消息的程序是生产者：

  ![A producer sends messages to a queue.](D:\sugon\file_markdown\部署文档\producer.png)

- 队列是 Rabbit MQ 中邮箱的名称。尽管消息流经 Rabbit MQ 和您的应用程序，但它们只能存储在队列中。

  队列仅受主机内存和磁盘限制的约束，它本质上是一个大型消息缓冲区。许多生产者可以将消息发送到一个队列，并且许多消费者可以尝试从一个队列接收数据。这就是我们表示队列的方式：

  ![A queue is the name for the post box in RabbitMQ.](https://www.rabbitmq.com/img/tutorials/queue.png)

- 消费与接收具有相似的含义。消费者是一个主要等待接收消息的程序：

  ![A consumer receives messages.](https://www.rabbitmq.com/img/tutorials/consumer.png)

请注意，生产者、消费者和代理不必驻留在同一主机上；事实上，在大多数应用中它们并不这样做。应用程序也可以既是生产者又是消费者。

## Hello World

（使用Java客户端）

在本教程的这一部分中，我们将用 Java 编写两个程序；发送单个消息的生产者和接收消息并将其打印出来的消费者。我们将忽略 Java API 中的一些细节，专注于这个非常简单的事情，以便开始。这是一个“Hello World”消息传递。

在下图中，“P”是我们的生产者，“C”是我们的消费者。中间的框是一个队列 - Rabbit MQ 代表消费者保留的消息缓冲区。

![(P) -> [|||] -> (C)](https://www.rabbitmq.com/img/tutorials/python-one.png)



> Java 客户端库
>
> Rabbit MQ 支持多种协议。本教程使用 AMQP 0-9-1，它是一种开放的通用消息传递协议。Rabbit MQ 有许多不同语言的客户端。我们将使用 Rabbit MQ 提供的 Java 客户端。下载客户端库及其依赖项（SLF4 J API 和 SLF4 J Simple）。将这些文件与教程 Java 文件一起复制到您的工作目录中。请注意，SLF4 J Simple 对于教程来说已经足够了，但您应该在生产中使用 Logback 等成熟的日志库。
>
> （Rabbit MQ Java 客户端也在中央 Maven 存储库中，组 ID 为 com.rabbitmq，工件 ID 为 amqp-client。）

现在我们有了 Java 客户端及其依赖项，我们可以编写一些代码了:

### Sending

![(P) -> [|||]](https://www.rabbitmq.com/img/tutorials/sending.png)

我们将调用消息发布者（发送者）Send 和消息消费者（接收者）Recv。发布者将连接到 Rabbit MQ，发送一条消息，然后退出。

在Send.java中，我们需要导入一些类：

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
```

设置类名和队列名称：

```java
public class Send {
  private final static String QUEUE_NAME = "hello";
  public static void main(String[] argv) throws Exception {
      ...
  }
}
```

然后，创建连接服务器代码：

```java
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");
try (Connection connection = factory.newConnection();
     Channel channel = connection.createChannel()) {

}
```

Connection对socket连接进行了抽象，并为我们处理协议版本协商、认证等工作。在这里，我们连接到本地计算机上的 Rabbit MQ 节点 - 因此是本地主机。如果我们想连接到另一台机器上的节点，我们只需在此处指定其主机名或 IP 地址即可。

接下来，我们创建一个通道，这是大多数用于完成任务的 API 所在的位置。请注意，我们可以使用 try-with-resources 语句，因为 Connection 和 Channel 都实现了 java.lang.Auto Closeable。这样我们就不需要在代码中显式关闭它们。

为了发送，我们必须声明一个队列供我们发送；然后我们可以将消息发布到队列，所有这些都在 try-with-resources 语句中：

```java
channel.queueDeclare(QUEUE_NAME, false, false, false, null);
String message = "Hello World!";
channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
System.out.println(" [x] Sent '" + message + "'");
```

声明队列是幂等的 - 仅当队列尚不存在时才会创建它。消息内容是一个字节数组，因此您可以在那里编码任何您喜欢的内容。

> 无法发送消息？
>
> 如果这是您第一次使用 Rabbit MQ 并且没有看到“已发送”消息，那么您可能会摸不着头脑，想知道可能出了什么问题。也许代理启动时没有足够的可用磁盘空间（默认情况下它至少需要 200 MB 可用空间），因此拒绝接受消息。检查代理日志文件以确认并在必要时减少限制。配置文件文档将向您展示如何设置disk_free_limit。

### Receiving

这就是我们的出版商的工作。我们的消费者侦听来自 Rabbit MQ 的消息，因此与发布单个消息的发布者不同，我们将保持消费者运行以侦听消息并将其打印出来。

![[|||] -> (C)](https://www.rabbitmq.com/img/tutorials/receiving.png)



代码（在 Recv.java 中）与 Send 具有几乎相同的导入：

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;
```

我们将使用额外的 Deliver Callback 接口来缓冲服务器推送给我们的消息。设置与发布者相同；我们打开一个连接和一个通道，并声明我们要从中消费的队列。

请注意，这与发送发布到的队列相匹配。

```java
public class Recv {
  private final static String QUEUE_NAME = "hello";
  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.queueDeclare(QUEUE_NAME, false, false, false, null);
    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

  }
}
```

请注意，我们也在这里声明了队列。因为我们可能会在发布者之前启动消费者，所以我们希望在尝试使用队列中的消息之前确保队列存在。

为什么我们不使用 try-with-resource 语句来自动关闭通道和连接？通过这样做，我们只需让程序继续运行，关闭所有内容，然后退出！这会很尴尬，因为我们希望进程在消费者异步侦听消息到达时保持活动状态。

我们将告诉服务器将队列中的消息传递给我们。由于它将异步向我们推送消息，因此我们以对象的形式提供回调，该回调将缓冲消息，直到我们准备好使用它们。这就是传递回调子类的作用。

```java
DeliverCallback deliverCallback = (consumerTag, delivery) -> {
    String message = new String(delivery.getBody(), "UTF-8");
    System.out.println(" [x] Received '" + message + "'");
};
channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
```

## 验证

> 这小节参考官网即可。

您可以仅使用类路径上的 Rabbit MQ java 客户端来编译这两个版本。

我在这里的处理：

前提是需要安装rabbitmq，使用docker安装。

首先使用docker镜像安装，安装上去之后，可以正常使用，但是无法访问管理界面。原因是docker镜像应该选择Management版本：

![image-20230830112729761](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301127072.png)

或者直接使用docker desktop中的rabbitmq插件即可：

![image-20230830112807258](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301128828.png)

默认使用最新的版本。

账号密码guest / guest

> docker容器内可以通过执行rabbitmqctl list_queues 来查看所有的队列。

代码：

Send.java

```java
package org.example.demo;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.nio.charset.StandardCharsets;

/**
 * @Author yamon
 * @Date 2023-08-29 16:09
 * @Description
 * @Version 1.0
 */
public class Send {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello World cccc!";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
```

Recv.java

```java
package org.example.demo;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.nio.charset.StandardCharsets;

/**
 * @Author yamon
 * @Date 2023-08-29 17:01
 * @Description
 * @Version 1.0
 */
public class Recv {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" + message + "'");
        };
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
    }
}
```

验证方式：

先启动Recv消费者，消费者永远处于开启状态，然后开启Send生产者。

![image-20230830113137440](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301131164.png)

![image-20230830113150183](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301131662.png)

## 参考

[RabbitMQ tutorial - "Hello World!" ]([RabbitMQ tutorial - "Hello World!" — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-one-java.html))
