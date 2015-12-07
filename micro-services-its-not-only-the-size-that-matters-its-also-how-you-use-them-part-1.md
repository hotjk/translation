Microservices: It’s not (only) the size that matters, it’s (also) how you use them – part 1

本文翻译自 Jeppe Cramon 在 [tigerteam.dk](https://www.tigerteam.dk) 的博客文章，原文地址：

> https://www.tigerteam.dk/2014/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-1/

在 [SOA – Hierarchy or organic growth?](http://www.tigerteam.dk/2014/soa-hierarchy-or-organic-growth/) 亦文中谈到了分层 SOA 的问题，在 [SOA : synchronous communication, data ownership and coupling](http://www.tigerteam.dk/2014/soa-synchronous-communication-data-ownership-and-coupling/) 一文中我们提到了四个面向服务架构的原则，还特别关注了服务间双向通信时的服务边界和自治问题，有了这些背景，让我们开始本文的主题。

如文章标题所言，我们讨论的是 Microservices（微服务），微服务架构是对应 Monolithic（一体化）架构的另一种架构方案。就像 SOA 这个概念缺乏明确定义一样，微服务也没有一个明确的定义。目前大家能达成的共识是微服务应该是[小的](http://oredev.org/2013/wed-fri-conference/implementing-micro-service-architectures)，并且服务是可以独立部署的。很多经验法则说微服务应该在 10 到 100 行代码，我觉得用代码行数作为评判服务粒度的标准有点儿无厘头。

我们缺少一种准则来指导我们从业务范围和集成方式来设计微服务。没有这种思想，在混杂的业务中设计合适的微服务就像从谷糠中筛选麦子一样困难，并且很容易受到分层 SOA 反模式（下图）的诱惑，最终走到经验法则的老路上。

传统的分层 SOA 里的服务是微服务么？

![分层 SOA 也是微服务架构？](https://www.tigerteam.dk/wp-content/uploads/2014/01/Layered-SOA.png "分层 SOA 也是微服务架构？")

传统的分层面向对象代码里，实体/数据服务（Entity/Data Services）是大致上跟仓储/数据访问对象（Repository/Data Access Object）功能类似的一层很薄的服务，实体服务（Entity Service）就是关系型数据库之上包了一层薄薄的壳。依据使用的编程语言和框架的差异，仓储（Repository）被大约 10 行到 300 行代码包装成 REST 或 Web 服务。符合微服务代码行数评判规则。

任务服务（Task Services）是协调和编排多个 Entity Service 调用的薄服务。依据使用的框架和库不同以及数据转换的程度，Task Services 一般用 10 行到 1000 行代码来实现，也算符合微服务代码行数评判规则。

过程服务（Process Services）是用来协调多个 Task Services 或 Entity Services 调用的一层较薄的服务。典型情况下会处理一些数据转换、更新失败时的补偿、长时间运行的事务等问题。根据使用框架和库的差异，Process Services 一般在 100 行到 几千行代码，勉强符合微服务代码行数评判规则。

就像在 [SOA: synchronous communication, data ownership and coupling](http://www.tigerteam.dk/2014/soa-synchronous-communication-data-ownership-and-coupling/) 文中对双向通信的讨论，在分层 SOA 架构下，服务责任描述和服务自治有严重的问题，Process Service 和 Task Service 的错误补偿相当复杂，服务合同变更和服务的突发故障也是问题，通过网络方式调用多个服务的延迟时间（发起服务调用到收到应答的时间）也很长。

上面的例子中，没单就代码行数对微服务进行分类，我们可以武断的认定 Entity Service/Task Service/Process Service 都是微服务，这清晰的表达出使用代码行数来区分服务是不是微服务的做法多糟糕。

如果我们竭尽所能地分解服务为非常小的服务，使用双向调用时延迟时间将非常可怕。如果微服务的焦点放在服务的大小上而不是使用模式上，不难想象我们将有一个服务调用星状图，一个应用调用一个服务，服务接着调用一堆小的服务，这些服务接着调用另一堆服务，这种使用模式甚至还会造成服务的循环调用。

![微服务同步双向通信星状图。服务调用服务，然后被调用服务又调用其他服务，然后……](https://www.tigerteam.dk/wp-content/uploads/2014/02/Microservices_star.png "微服务同步双向通信星状图。服务调用服务，然后被调用服务又调用其他服务，然后……")

我们尝试分解服务到非常小的粒度，造成一个服务需要再调用很多服务才能完成自己的任务，这是因为他们都想要对方的数据和功能。

面向服务的一个目标是重用，怎么保证尽可能的重用？使所有的服务尽可能的小？进而让他们在不同的场景可以被复用？看起来逻辑是对的，但问题是，每次我们这样复用服务时也增加了耦合。面向服务的另一个目标是减少耦合，减少耦合才能在保持商业和技术的要求的前提下很容易的对服务进行修改。

在 [SOA: synchronous communication, data ownership and coupling](http://www.tigerteam.dk/2014/soa-synchronous-communication-data-ownership-and-coupling/) 一文中，我们讨论了同步双向通信导致的一些复杂的耦合令人绝望：

- 通信相关的耦合（数据和服务不总是在一个服务内）
- 分层耦合（业务和安全、持久化等不在一个服务内）
- 时间耦合（当服务依赖的其他服务无法访问时服务不可用）

耦合有导致级联副作用的倾向。当修改服务协议时，所用使用该服务的其他服务都要跟着修改。当服务不可用时，所有依赖于该服务的其他服务也不可用了。当一个服务更新数据失败，所在 Process Services 的其他服务也需要应对更新失败的影响。

![错误是在服务收到消息执行更新操作前发生的还是更新操作后发生的？](http://www.tigerteam.dk/wp-content/uploads/2014/02/Service-call-failing.png “错误是在服务收到消息执行更新操作前发生的还是更新操作后发生的？”)

上图中从客户端发起服务的调用，或者从服务中调用另一个服务，因为使用双向通信，客户端发送一个请求消息给服务，服务收到请求并执行一些处理，比如更新数据库，处理完成后，服务发送一个应答消息给客户端表明处理的结果。通信经由网络，比如 HTTP 调用或者 Queue 消息。比起我们在 Monolithic 开发方式下的内存方法调用，网络请求是缓慢且不稳定的。

如果客户端收到超时应答或者网络 IO 错误，典型情况有两种可能：

-  请求没有到达服务，所以数据库没有被更新。
-  服务收到了请求且更新了数据库，但是客户端没收到应答。

缺少可靠的消息机制意味着客户端不知道服务做了还是没做，这给客户端留下一个难题，客户端要怎么做呢：

-  尝试问服务是否处理了请求，如果没处理重新发起调用？
-  还是草率的直接发起重试？
-  是启动补偿？
-  还是放弃调用？

如果客户端尝试再次调用，服务必须实现允许对同一笔业务多次调用，或者接受同一笔业务相同的请求消息，仍然保证业务只被执行一次，也就是说服务必须是[幂等的](http://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning)。
如果要对多个服务进行一系列更新操作，我们必须面对一个巨大的一致性问题，因为我们没有也不应该使用分布式事务来协调这些更新。就像我们在 [SOA: synchronous communication, data ownership and coupling](http://www.tigerteam.dk/2014/soa-synchronous-communication-data-ownership-and-coupling/) 一文中谈到的，双向通信的补偿逻辑既不是微不足道的也不是简单的。

![同步双向通信方式集成事务补偿](http://www.tigerteam.dk/wp-content/uploads/2014/02/synchronous-SOA-orchestration.png "同步双向通信方式集成事务补偿")

假设一种极端的微服务架构，每个服务只负责实体的一个属性（比如姓名、地址、邮编、城市等等），延迟时间会是一个巨大的问题，稳定性、协调服务调用和补偿也会是巨大的问题。我们一定是没找到正确的服务设计指导准则。

下一篇文章我们将谈到怎么在分布式环境集成服务，并看一下服务粒度对集成的影响。