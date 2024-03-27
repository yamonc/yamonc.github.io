---
title: RabbitMQ官方教程-Work queues
date: 2023-08-30 15:56:23
tags: [RabbitMQ]
categories: [消息队列]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301556033.jpeg
comment: 是
keywords: 
password: 
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# RabbitMQ官方教程-Work queues

![a river running through a canyon](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301556033.jpeg)

## Work Queues

![Producer -> Queue -> Consuming: Work Queue used to distribute time-consuming tasks among multiple workers.](D:\blog\source\_posts\12. RabbitMQ\assets\python-two.png)

在第一个教程中，我们编写了从命名队列发送和接收消息的程序。在本教程中，我们将创建一个*Work Queue* ，用于在多个worker之间分配耗时的任务。

工作队列（又名：任务队列）背后的主要思想是避免立即执行资源密集型任务并必须等待其完成。相反，我们安排稍后完成的任务。我们将任务封装为消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行作业。

当您运行许多worker时，任务将在他们之间共享。

这个概念在 Web 应用程序中特别有用，因为在 Web 应用程序中不可能在较短的 HTTP 请求窗口内处理复杂的任务。

## 准备

在本教程的前一部分中，我们发送了一条包含“Hello World!”的消息。现在我们将发送代表复杂任务的字符串。

我们没有现实世界的任务，比如要调整图像大小或要渲染 pdf 文件，所以让我们通过使用 Thread.sleep() 函数假装我们很忙来伪造它。我们将字符串中点数作为其复杂度；每个点将占一秒钟的“工作”。例如，Hello... 描述的一个假任务将需要三秒钟。

我们将稍微修改前面示例中的 Send.java 代码，以允许从命令行发送任意消息。该程序会将任务调度到我们的工作队列中，因此我们将其命名为 New Task.java：

```java
String message = String.join(" ", argv);

channel.basicPublish("", "hello", null, message.getBytes());
System.out.println(" [x] Sent '" + message + "'");
```

我们旧的 Recv.java 程序还需要一些更改：它需要为消息正文中的每个点伪造一秒钟的工作。它将处理传递的消息并执行任务，所以我们将其称为 Worker.java：

```java
DeliverCallback deliverCallback = (consumerTag, delivery) -> {
  String message = new String(delivery.getBody(), "UTF-8");

  System.out.println(" [x] Received '" + message + "'");
  try {
    doWork(message);
  } finally {
    System.out.println(" [x] Done");
  }
};
boolean autoAck = true; // acknowledgment is covered below
channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, consumerTag -> { });
```

我们用假的任务来模拟执行时间：

```java
private static void doWork(String task) throws InterruptedException {
    for (char ch: task.toCharArray()) {
        if (ch == '.') Thread.sleep(1000);
    }
}
```

按照教程一中的方式编译它们（使用工作目录中的 jar 文件和环境变量 CP）：

```bash
javac -cp $CP NewTask.java Worker.java
```

## 轮询分发

使用任务队列的优点之一是能够轻松并行工作。如果我们正在积压工作，我们可以添加更多worker，这样就可以轻松扩展。

首先，让我们尝试同时运行两个工作实例。他们都会从队列中获取消息，但是具体如何获取呢？让我们来看看。

您需要打开三个控制台。两个将运行工人程序。这些控制台将是我们的两个消费者 - C1 和 C2。

```bash
# shell 1
java -cp $CP Worker
# => [*] Waiting for messages. To exit press CTRL+C
# shell 2
java -cp $CP Worker
# => [*] Waiting for messages. To exit press CTRL+C
```

在第三个任务中，我们将发布新任务。启动消费者后，您可以发布一些消息：

```bash
# shell 3
java -cp $CP NewTask First message.
# => [x] Sent 'First message.'
java -cp $CP NewTask Second message..
# => [x] Sent 'Second message..'
java -cp $CP NewTask Third message...
# => [x] Sent 'Third message...'
java -cp $CP NewTask Fourth message....
# => [x] Sent 'Fourth message....'
java -cp $CP NewTask Fifth message.....
# => [x] Sent 'Fifth message.....'
```

给worker传输了什么消息：

```bash
java -cp $CP Worker
# => [*] Waiting for messages. To exit press CTRL+C
# => [x] Received 'First message.'
# => [x] Received 'Third message...'
# => [x] Received 'Fifth message.....'
```

```bash
java -cp $CP Worker
# => [*] Waiting for messages. To exit press CTRL+C
# => [x] Received 'Second message..'
# => [x] Received 'Fourth message....'
```

默认情况下，Rabbit MQ 会将每条消息按顺序发送给下一个消费者。平均而言，每个消费者都会收到相同数量的消息。这种分发消息的方式称为循环法。与三名或更多工人一起尝试此操作。

## 消息确认

执行一项任务可能需要几秒钟的时间，您可能想知道如果消费者启动一项长任务并在完成之前终止会发生什么。

使用我们当前的代码，一旦 Rabbit MQ 将消息传递给消费者，它会立即将其标记为删除。在这种情况下，如果终止一个工作线程，它刚刚处理的消息就会丢失。已发送给该特定worker但尚未处理的消息也会丢失。

但我们不想失去任何任务。如果一个worker失效，我们希望将任务交付给另一个worker。

为了确保消息永远不会丢失，Rabbit MQ 支持消息确认。消费者发回确认消息，告诉 Rabbit MQ 已收到并处理特定消息，并且 Rabbit MQ 可以自由删除该消息。

如果消费者在没有发送 ack 的情况下死亡（其通道关闭、连接关闭或 TCP 连接丢失），Rabbit MQ 将了解消息未完全处理并将重新排队。如果同时有其他消费者在线，那么它会快速将其重新传递给另一个消费者。这样你就可以确保不会丢失任何消息，即使worker偶尔会down掉。

消费者交付确认时强制执行超时（默认为 30 分钟）。这有助于检测从不确认交付的有问题（卡住）的消费者。您可以按照传送确认超时中的说明增加此超时。

默认情况下，手动消息确认处于打开状态。在前面的示例中，我们通过 autoAck=true 标志显式关闭它们。当我们完成任务后，是时候将此标志设置为 false 并向worker发送适当的确认。

```java
channel.basicQos(1); // accept only one unack-ed message at a time (see below)

DeliverCallback deliverCallback = (consumerTag, delivery) -> {
  String message = new String(delivery.getBody(), "UTF-8");

  System.out.println(" [x] Received '" + message + "'");
  try {
    doWork(message);
  } finally {
    System.out.println(" [x] Done");
    channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
  }
};
boolean autoAck = false;
channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, consumerTag -> { });
```

使用此代码，您可以确保即使在处理消息时使用 CTRL+C 终止工作程序，也不会丢失任何内容。工作线程终止后不久，所有未确认的消息都会被重新传递。

确认必须在接收交付的同一通道上发送。尝试使用不同的通道进行确认将导致通道级协议异常。请参阅有关确认的文档指南以了解更多信息。

> 忘记确认？
>
> 忘记添加basicAck 是一个常见的错误。这是一个很容易犯的错误，但后果却很严重。当您的客户端退出时，消息将被重新传送（这可能看起来像随机重新传送），但 Rabbit MQ 会占用越来越多的内存，因为它无法释放任何未确认的消息。
>
> 为了调试这种错误，您可以使用rabbitmqctl打印messages_unacknowledged字段：
>
> ```bash
> sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged
> ```

## 消息持久化

我们已经学会了如何确保即使消费者死亡，任务也不会丢失。但是如果 Rabbit MQ 服务器停止，我们的任务仍然会丢失。

当 Rabbit MQ 退出或崩溃时，它会忘记队列和消息，除非您告诉它不要这样做。要确保消息不丢失，需要做两件事：我们需要将队列和消息标记为持久的。

首先，我们需要确保队列能够在 Rabbit MQ 节点重新启动后继续存在。为此，我们需要将其声明为持久的：

```java
boolean durable = true;
channel.queueDeclare("hello", durable, false, false, null);
```

虽然这个命令本身是正确的，但它在我们当前的设置中不起作用。这是因为我们已经定义了一个名为 hello 的队列，它是不持久的。Rabbit MQ 不允许您使用不同的参数重新定义现有队列，并将向任何尝试执行此操作的程序返回错误。但是有一个快速的解决方法 - 让我们声明一个具有不同名称的队列，例如task_queue：

```java
boolean durable = true;
channel.queueDeclare("task_queue", durable, false, false, null);
```

此队列声明更改需要应用于生产者和消费者代码。此时我们就可以确定，即使RabbitMQ重启，task_queue队列也不会丢失。现在我们需要将消息标记为持久消息 - 通过将消息属性（实现基本属性）设置为值 PERSISTENT_TEXT_PLAIN。

```java
import com.rabbitmq.client.MessageProperties;

channel.basicPublish("", "task_queue",
            MessageProperties.PERSISTENT_TEXT_PLAIN,
            message.getBytes());
```

> 关于消息持久化的注意事项
>
> 将消息标记为持久并不能完全保证消息不会丢失。尽管它告诉 Rabbit MQ 将消息保存到磁盘，但 Rabbit MQ 已接受消息但尚未保存的时间窗口仍然很短。此外，Rabbit MQ 不会对每条消息执行 fsync(2) —— 它可能只是保存到缓存中，而不是真正写入磁盘。持久性保证并不强，但对于我们简单的任务队列来说已经足够了。
>
> 如果您需要更强的保证，那么您可以使用发布者确认（publisher confirms）。

## 公平调度

您可能已经注意到，调度仍然没有完全按照我们想要的方式工作。例如，在有两名工作人员的情况下，当所有奇数消息都很重而偶数消息都很轻时，一名工作人员将一直忙碌，而另一名工作人员几乎不会做任何工作。

好吧，Rabbit MQ 对此一无所知，仍然会均匀地分发消息。

发生这种情况是因为 Rabbit MQ 只是在消息进入队列时调度消息。它不会查看消费者未确认消息的数量。它只是盲目地将每条第 n 条消息分派给第 n 个消费者。

![Producer -> Queue -> Consuming: RabbitMQ dispatching messages.](D:\blog\source\_posts\12. RabbitMQ\assets\prefetch-count.png)

为了解决这个问题，我们可以使用基本的 Qos 方法和预取计数 = 1 设置。这告诉 Rabbit MQ 不要一次向一个工作线程发送多于一条消息。或者，换句话说，在工作人员处理并确认前一条消息之前，不要向工作人员发送新消息。

相反，它会将其分派给下一个不忙的工作人员。

```java
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```

> 关于队列大小的注意事项
>
> 如果所有工作人员都很忙，您的队列可能会被填满。您需要密切关注这一点，也许添加更多的worker，或者制定其他策略。

## 实践

NewTask.java

```java
package org.example.demo1;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

/**
 * @Author yamon
 * @Date 2023-08-29 17:53
 * @Description
 * @Version 1.0
 */
public class NewTask {
    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);

            String message = String.join(" ", argv);

            channel.basicPublish("", TASK_QUEUE_NAME,
                    MessageProperties.PERSISTENT_TEXT_PLAIN,
                    message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
```

Worker.java

```java
package org.example.demo1;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

/**
 * @Author yamon
 * @Date 2023-08-30 10:01
 * @Description
 * @Version 1.0
 */
public class Worker {
    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        channel.basicQos(1);

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");

            System.out.println(" [x] Received '" + message + "'");
            try {
                doWork(message);
            } finally {
                System.out.println(" [x] Done");
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };
        channel.basicConsume(TASK_QUEUE_NAME, false, deliverCallback, consumerTag -> { });
    }

    private static void doWork(String task) {
        for (char ch : task.toCharArray()) {
            if (ch == '.') {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException _ignored) {
                    Thread.currentThread().interrupt();
                }
            }
        }
    }
}
```

将Worker设置成为多启动模式。

idea设置启动方式：

![image-20230830140112359](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301401478.png)

![image-20230830140133591](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301401147.png)

总体流程：

开启三个worker控制台，执行三次newTask任务，会发现，在每个worker控制台按顺序收到一条消息：

![image-20230830140224905](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308301402386.png)

使用消息确认和prefetchCount ，您可以设置工作队列。

有关Channel方法和MessageProperties的更多信息，您可以在线浏览 Java 文档。

即使 Rabbit MQ 重新启动，持久性选项也能让任务继续存在。

现在我们可以继续教程 3，学习如何向许多消费者传递相同的消息。
