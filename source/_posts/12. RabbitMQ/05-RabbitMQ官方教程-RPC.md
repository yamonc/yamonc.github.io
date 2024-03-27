---
title: 05-RabbitMQ官方教程-RPC
date: 2023-08-31 15:56:52
tags: [RabbitMQ]
categories: [消息队列]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308311556346.jpeg
comment: true
keywords: 
password: 
abstract: 有东西被加密了, 请输入密码查看.
message: 您好, 这里需要密码.
wrong_pass_message: 抱歉, 这个密码看着不太对, 请再试试.
wrong_hash_message: 抱歉, 这个文章不能被校验, 不过您还是能看看解密后的内容.
---
# 05-RabbitMQ官方教程-RPC

![white mountain](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308311556346.jpeg)

远程过程调用RPC

在第二个教程中，我们学习了如何使用work queues在多个不同worker之间分配耗时的任务。

但是如果我们需要在远程计算机上运行一个函数并等待结果怎么办？

嗯，那是一个不同的故事。这种模式通常称为远程过程调用或 RPC。

在本教程中，我们将使用 Rabbit MQ 构建一个 RPC 系统：一个客户端和一个可扩展的 RPC 服务器。

由于我们没有任何值得分发的耗时任务，因此我们将创建一个返回斐波那契数的虚拟 RPC 服务。

## 客户端接口

为了说明如何使用 RPC 服务，我们将创建一个简单的客户端类。

它将公开一个名为 call 的方法，该方法发送 RPC 请求并阻塞，直到收到答案：

```java
FibonacciRpcClient fibonacciRpc = new FibonacciRpcClient();
String result = fibonacciRpc.call("4");
System.out.println( "fib(4) is " + result);
```

> 关于RPC笔记：
>
> 尽管 RPC 是计算中非常常见的模式，但它经常受到批评。当程序员不知道函数调用是本地函数还是慢速 RPC 时，就会出现问题。类似的混乱会导致系统不可预测，并给调试增加不必要的复杂性。滥用 RPC 不但不会简化软件，反而会导致难以维护的意大利面条式代码。
>
> 考虑到这一点，请考虑以下建议：
>
> - 确保哪个函数调用是本地的、哪个是远程的很明显。
>
> - 记录您的系统。明确组件之间的依赖关系。
>
> - 处理错误情况。当RPC服务器长时间宕机时，客户端应该如何反应？
>
> 如有疑问，请避免使用 RPC。如果可以的话，您应该使用异步管道 - 而不是类似 RPC 的阻塞，而是将结果异步推送到下一个计算阶段。

## 回调队列（Callback queue）

一般来说，通过 Rabbit MQ 进行 RPC 很容易。客户端发送请求消息，服务器回复响应消息。为了接收响应，我们需要随请求发送“回调”队列地址。我们可以使用默认队列（Java客户端独有）。我们来尝试一下：

```java
callbackQueueName = channel.queueDeclare().getQueue();

BasicProperties props = new BasicProperties
                            .Builder()
                            .replyTo(callbackQueueName)
                            .build();

channel.basicPublish("", "rpc_queue", props, message.getBytes());

// ... then code to read a response message from the callback_queue ...
```

新的导入：

```java
import com.rabbitmq.client.AMQP.BasicProperties;
```

> 消息属性：
>
> AMQP 0-9-1 协议预定义了消息附带的一组 14 个属性。大多数属性很少使用，但以下属性除外：
>
> 传递模式（deliveryMode）：将消息标记为持久（值为 2）或瞬态（任何其他值）。您可能还记得第二个教程中的这个属性。
>
> 内容类型（contentType）：用于描述编码的 mime 类型。例如，对于经常使用的 JSON 编码，最好将此属性设置为：application/json。
>
> 回复（replyTo）：通常用于命名回调队列。
>
> 关联 ID（correlationId）：用于将 RPC 响应与请求关联起来。

## Correlation Id（相关ID）

在上面介绍的方法中，我们建议为每个 RPC 请求创建一个回调队列。这是相当低效的，但幸运的是有一个更好的方法 - 让我们为每个客户端创建一个回调队列。

这引发了一个新问题，在该队列中收到响应后，不清楚该响应属于哪个请求。这就是使用correlationId属性的时候。

我们将为每个请求将其设置为唯一值。稍后，当我们在回调队列中收到消息时，我们将查看此属性，并基于此我们将能够将响应与请求进行匹配。如果我们看到未知的correlationId 值，我们可以安全地丢弃该消息 - 它不属于我们的请求。

您可能会问，为什么我们应该忽略回调队列中的未知消息，而不是因错误而失败？这是由于服务器端可能存在竞争条件。虽然不太可能，但 RPC 服务器有可能在向我们发送答案之后、发送请求的确认消息之前就挂掉了。如果发生这种情况，重新启动的 RPC 服务器将再次处理该请求。这就是为什么在客户端我们必须优雅地处理重复的响应，并且 RPC 理想情况下应该是幂等的。

## 概要

![Summary illustration, which is described in the following bullet points.](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202308311542203.png)

我们的RPC工作如下：

- 对于 RPC 请求，客户端发送一条具有两个属性的消息：reply To（回复到），它被设置为专门为该请求创建的匿名独占队列；以及correlation Id（它被设置为每个请求的唯一值）。请求被发送到 rpc_queue 队列。
- 请求被发送到 rpc_queue 队列。
- RPC 工作线程（又名：服务器）正在等待该队列上的请求。当出现请求时，它会执行作业并使用replyTo字段中的队列将带有结果的消息发送回客户端。
- 客户端等待回复队列上的数据。当出现消息时，它会检查相关 Id 属性。如果它与请求中的值匹配，它将向应用程序返回响应。

## 组装

斐波那契任务：

```java
private static int fib(int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    return fib(n-1) + fib(n-2);
}
```

我们声明我们的斐波那契函数。它假设仅有效的正整数输入。（不要指望这个可以处理大数，而且它可能是最慢的递归实现）。

我们的 RPC 服务器的代码可以在这里找到：RPCServer.java。

服务器代码相当简单：

- 像往常一样，我们首先建立连接、通道并声明队列。

- 我们可能想要运行多个服务器进程。为了将负载均匀地分布在多个服务器上，我们需要在channel.basic Qos中设置prefetchCount 。

- 我们使用basicConsume 来访问队列，其中我们以对象的形式提供回调（Deliver Callback），该回调将完成工作并将响应发回。

我们的 RPC 客户端的代码可以在这里找到：RPCClient.java。

客户端代码稍微复杂一些：

- 我们建立了connection和channel。

- 我们的call方法发出实际的RPC请求。

- 在这里，我们首先生成一个唯一的关联Id号并保存它——我们的消费者回调将使用这个值来匹配适当的响应。

- 然后，我们为回复创建一个专用的独占队列并订阅它。

- 接下来，我们发布具有两个属性的请求消息：reply-To和correlation-Id。

- 在这一点上，我们可以坐下来等待，直到适当的回应到来。

- 由于我们的消费者交付处理是在一个单独的线程中进行的，因此我们需要在响应到达之前挂起主线程。使用Completable Future是一种可能的解决方案。

- 消费者正在做一项非常简单的工作，对于每一条消费的响应消息，它都会检查相关性Id是否是我们要查找的。如果是这样，它就完成了复杂的未来。

- 同时，主线程正在等待Completable Future完成。

- 最后，我们将响应返回给用户。

代码：

```java
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.UUID;
import java.util.concurrent.*;

public class RPCClient implements AutoCloseable {

    private Connection connection;
    private Channel channel;
    private String requestQueueName = "rpc_queue";

    public RPCClient() throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");

        connection = factory.newConnection();
        channel = connection.createChannel();
    }

    public static void main(String[] argv) {
        try (RPCClient fibonacciRpc = new RPCClient()) {
            for (int i = 0; i < 32; i++) {
                String i_str = Integer.toString(i);
                System.out.println(" [x] Requesting fib(" + i_str + ")");
                String response = fibonacciRpc.call(i_str);
                System.out.println(" [.] Got '" + response + "'");
            }
        } catch (IOException | TimeoutException | InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }

    public String call(String message) throws IOException, InterruptedException, ExecutionException {
        final String corrId = UUID.randomUUID().toString();

        String replyQueueName = channel.queueDeclare().getQueue();
        AMQP.BasicProperties props = new AMQP.BasicProperties
                .Builder()
                .correlationId(corrId)
                .replyTo(replyQueueName)
                .build();

        channel.basicPublish("", requestQueueName, props, message.getBytes("UTF-8"));

        final CompletableFuture<String> response = new CompletableFuture<>();

        String ctag = channel.basicConsume(replyQueueName, true, (consumerTag, delivery) -> {
            if (delivery.getProperties().getCorrelationId().equals(corrId)) {
                response.complete(new String(delivery.getBody(), "UTF-8"));
            }
        }, consumerTag -> {
        });

        String result = response.get();
        channel.basicCancel(ctag);
        return result;
    }

    public void close() throws IOException {
        connection.close();
    }
}
```



```java
import com.rabbitmq.client.*;

public class RPCServer {

    private static final String RPC_QUEUE_NAME = "rpc_queue";

    private static int fib(int n) {
        if (n == 0) return 0;
        if (n == 1) return 1;
        return fib(n - 1) + fib(n - 2);
    }

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare(RPC_QUEUE_NAME, false, false, false, null);
        channel.queuePurge(RPC_QUEUE_NAME);

        channel.basicQos(1);

        System.out.println(" [x] Awaiting RPC requests");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            AMQP.BasicProperties replyProps = new AMQP.BasicProperties
                    .Builder()
                    .correlationId(delivery.getProperties().getCorrelationId())
                    .build();

            String response = "";
            try {
                String message = new String(delivery.getBody(), "UTF-8");
                int n = Integer.parseInt(message);

                System.out.println(" [.] fib(" + message + ")");
                response += fib(n);
            } catch (RuntimeException e) {
                System.out.println(" [.] " + e);
            } finally {
                channel.basicPublish("", delivery.getProperties().getReplyTo(), replyProps, response.getBytes("UTF-8"));
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };

        channel.basicConsume(RPC_QUEUE_NAME, false, deliverCallback, (consumerTag -> {}));
    }
}
```

