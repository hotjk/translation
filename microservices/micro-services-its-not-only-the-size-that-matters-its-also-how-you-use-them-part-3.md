Microservices: It’s not (only) the size that matters, it’s (also) how you use them – part 3

[第一部分](https://github.com/hotjk/translation/blob/master/microservices/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-1.md)
[第二部分](https://github.com/hotjk/translation/blob/master/microservices/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-2.md)
[第三部分](https://github.com/hotjk/translation/blob/master/microservices/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-3.md)
[第四部分](https://github.com/hotjk/translation/blob/master/microservices/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-4.md)
[第五部分](https://github.com/hotjk/translation/blob/master/microservices/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-5.md)

本文翻译自 Jeppe Cramon 在 [tigerteam.dk](https://www.tigerteam.dk) 的博客文章，原文地址：

> https://www.tigerteam.dk/2014/microservices-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-3/

前文中，我们再次描述服务（微服务）之间进行双向同步通信的问题，我们讨论了微服务使用双向同步通信造成的耦合问题实际上是分布式对象的一种变形。我们还谈及这种缺乏可信任的消息和事务导致服务失败时复杂的补偿逻辑。

重温了八个分布式计算谬论后，我们开始寻找双向通信调用服务的替换方案，从 Pat Hellands 的 [Life Beyond Distributed Transactions ? – An Apostate ‘s Opinion](http://www-db.cs.wisc.edu/cidr/cidr2007/papers/cidr07p15.pdf) 一文中我们看到分布式事务并不是协调服务更新操作的解决方案，分布式事务是有问题的。

根据 Pat Hellands 所言，我们必须解决一下问题：

1. 怎么拆分数据和服务
2. 怎样发现数据和服务
3. 数据服务间如何通信

前两个问题我们在前文中已经谈及，可以归纳为：

- 数据必须被组织成实体或聚合（DDD 术语）
- 聚合必须有唯一可标识的 ID（比如 UUID/GUID）
- 聚合需要限定大小以保证事务一致性
- 经验法则是：1 usercase = 1 transaction = 1 aggregate

下面我们继续说第三个问题

## 数据服务间如何通信

前面说了多次，服务间使用双向同步通信导致深度耦合和其他的讨厌的地方：

- 导致通信相关的耦合（因为数据和逻辑不总在一个服务内），也导致服务合同、数据、功能耦合以及网络通信的高延迟时间
- 分层耦合（持久化不总在一个服务内）
- 时间耦合（当服务依赖的其他服务无法访问时服务不可用）
- 服务依赖其他服务减少了自身的自治能力，也让服务更不可靠
- 所有这些导致在没有可靠消息和事务的情况下复杂的逻辑补偿

![服务复用在双向同步通信方式下的耦合](http://www.tigerteam.dk/wp-content/uploads/2014/03/reusable-services-and-coupling.png "服务复用在双向同步通信方式下的耦合")

### 如果同步通信不是办法，那么是不是应该用异步通信？

是的，但是取决于…… 说取决于什么之前，我们先看看同步和异步通信的特点：

- 同步通信需要消费者和提供者通信发生时都在线，比如载入一个网页或者两个人对话。
- 异步通信的发送者和接收者不需要同时在线，异步通信方式打破了发送者和接收者之间的临时耦合，比如电子邮件、短信、邮寄。

基于这些特点，我们简单的对通信进行了分类：

### 同步通信是双向通信

![同步通信](http://www.tigerteam.dk/wp-content/uploads/2014/04/synchronous-communication.png "同步通信")

上图的同步通信方式称为 Request/Response 模式，典型情况下以 PRC 模式实现。
在 Request/Response 模式下，服务消费者发送一个请求消息给服务提供者，在服务提供者处理请求消息时，服务消费者基本上只能等待直到收到应答或者发生错误（有些人可能会说，消费者可以利用异步平台特性，在等待时并行的做其他调用，但这其实没有解决调用时消费者和提供者的时间耦合，在收到应答前消费者根本无法继续后面的工作）。典型的 Request/Response 或者 RPC 调用流程如下图：

![同步通信流程](http://www.tigerteam.dk/wp-content/uploads/2014/04/Sync-waits.png "同步通信流程")

如图所示，服务消费者和服务提供者之间是强耦合的，服务提供者不可用时，服务消费者也不能工作，这类时间耦合或者说运行时耦合是我们应该尽量避免的。

### 异步通信是单向通信

![异步通信](http://www.tigerteam.dk/wp-content/uploads/2014/04/asynchronous-communication.png "异步通信")

使用异步通信时，发送者通过传输通道发出一个消息给接收者，发送者只需要短暂的等待就可以获得消息送达的反馈，然后发送者就可以继续后面的工作。这就是单向通信的本质，典型的执行路径如下图：

![异步通信流程](http://www.tigerteam.dk/wp-content/uploads/2014/04/async-waits.png "异步通信流程")

异步通信就是大家常说的消息传送，异步通信的传输通道负责接受发送者发来的消息，并负责把消息交付给接收者（可能有多个）。可以说传输通道承担了消息交换的职责，传输通道可以非常简单（比如使用 Socket、0MQ），也可以使用带有持久化的高级分布式解决方案（比如 ActiveMQ、HornetQ、kafka 等等）。消息机制和异步通信通道提供了不同消息交换保障：交付保证和消息顺序保证。

### 为什么异步通信不是完美的解决方案？

事实上，很遗憾，异步通信不像同步通信一样容易，服务的集成模式决定了真实的耦合程度。

双向通信有几种形式或者变化：

- 远程过程调用（RPC）-- 异步通信
- Request/Response  -- 同步通信
- Request/Reply  -- 也被称为基于异步的同步通信。

经常看到一些项目使用 Request/Reply （基于异步的同步通信）来确保服务不会产生时间耦合。说到细节就比较讨厌了，从消费者来看，大多数使用 Request/Reply 模式的应用程序，时间耦合的程度跟使用 RPC 或者 Request/Response 没有太大的区别，他们统统是双向通信的变种，都要求发送者等到应答后才能继续处理业务。

![RPC 和 Request/Response -- 同步双向通信 VS. Request/Reply -- 异步双向通信](http://www.tigerteam.dk/wp-content/uploads/2014/04/RPC-Request-Response-vs-Request-Reply.png "RPC 和 Request/Response -- 同步双向通信 VS. Request/Reply -- 异步双向通信")

### 那么结论是？

结论就是服务间双向通信是问题的根源，我们把服务粒度变小（变成微服务），这个问题也不会变小。
我们已经看到异步通信可以打破服务间的时间耦合，但是只有这种异步通信是单向时才真正起作用。

![我们的方案:)](http://www.tigerteam.dk/wp-content/uploads/2014/04/asynchronous-communication.png "我们的方案:)")

问题是怎样在只使用服务间异步单向通信的限制下设计你的服务或微服务（UI 和服务之间通信时另一件事，我们后面会谈到）？

下一篇我们看看如何拆分服务以及怎样让通过异步单行通信调用它们。
