﻿Microservices: It’s not (only) the size that matters, it’s (also) how you use them

本文翻译自 Jeppe Cramon 在 [tigerteam.dk](https://www.tigerteam.dk) 的博客文章，原文地址：

> https://www.tigerteam.dk/2014/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-1/
> https://www.tigerteam.dk/2014/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-2/
> https://www.tigerteam.dk/2014/microservices-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-3/
> https://www.tigerteam.dk/2014/microservices-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-4/
> https://www.tigerteam.dk/2015/microservices-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-5/

在 [SOA – Hierarchy or organic growth?](http://www.tigerteam.dk/2014/soa-hierarchy-or-organic-growth/) 一文中谈到了分层 SOA 的问题，在 [SOA : synchronous communication, data ownership and coupling](http://www.tigerteam.dk/2014/soa-synchronous-communication-data-ownership-and-coupling/) 一文中我们提到了四个面向服务架构的原则，还特别关注了服务间双向通信时的服务边界和自治问题，有了这些背景，让我们开始本文的主题。

如文章标题所言，我们讨论的是 Microservices（微服务），微服务架构是对应 Monolithic（一体化）架构的另一种架构方案。就像 SOA 这个概念缺乏明确定义一样，微服务也没有一个明确的定义。目前大家能达成的共识是微服务应该是[小的](http://oredev.org/2013/wed-fri-conference/implementing-micro-service-architectures)，并且服务是可以独立部署的。很多经验法则说微服务应该在 10 到 100 行代码，我觉得用代码行数来作为评判服务粒度的标准有点儿恐怖。

我们缺少一种准则来指导我们从业务范围和集成方式来设计微服务。没有这种思想，在混杂的业务中设计合适的微服务就像从谷糠中筛选麦子一样困难，并且很容易受到分层 SOA 反模式（下图）的诱惑，最终走到经验法则的老路上。

传统的分层 SOA 里的服务是微服务么？

![分层 SOA 也是微服务架构？](https://github.com/hotjk/translation/blob/master/microservices/Image/Layered-SOA.png?raw=true)

传统的分层面向对象代码里，实体/数据服务（Entity/Data Services）是大致上跟仓储/数据访问对象（Repository/Data Access Object）功能类似的一层很薄的服务，实体服务（Entity Service）就是关系型数据库之上包了一层薄薄的壳。依据使用的编程语言和框架的差异，仓储（Repository）被大约 10 行到 300 行代码包装成 REST 或 Web 服务。符合微服务代码行数评判规则。

任务服务（Task Services）是协调和编排多个 Entity Service 调用的薄服务。依据使用的框架和库不同以及数据转换的程度，Task Services 一般用 10 行到 1000 行代码来实现，也算符合微服务代码行数评判规则。

过程服务（Process Services）是用来协调多个 Task Services 或 Entity Services 调用的一层较薄的服务。典型情况下会处理一些数据转换、更新失败时的补偿、长时间运行的事务等问题。根据使用框架和库的差异，Process Services 一般在 100 行到 几千行代码，勉强符合微服务代码行数评判规则。

就像在 [SOA: synchronous communication, data ownership and coupling](http://www.tigerteam.dk/2014/soa-synchronous-communication-data-ownership-and-coupling/) 文中对双向通信的讨论，在分层 SOA 架构下，服务责任描述和服务自治有严重的问题，Process Service 和 Task Service 的错误补偿相当复杂，服务合同变更和服务的突发故障也是问题，通过网络方式调用多个服务的延迟时间（发起服务调用到收到应答的时间）也很长。

上面的例子中，没单就代码行数对微服务进行分类，我们可以武断的认定 Entity Service/Task Service/Process Service 都是微服务，这清晰的表达出使用代码行数来区分服务是不是微服务的做法多糟糕。

如果我们竭尽所能地分解服务为非常小的服务，使用双向调用时延迟时间将非常可怕。如果微服务的焦点放在服务的大小上而不是使用模式上，不难想象我们将有一个服务调用星状图，一个应用调用一个服务，服务接着调用一堆小的服务，这些服务接着调用另一堆服务，这种使用模式甚至还会造成服务的循环调用。

![微服务同步双向通信星状图。服务调用服务，然后被调用服务又调用其他服务，然后……](https://github.com/hotjk/translation/blob/master/microservices/Image/Microservices_star.png?raw=true)

我们尝试分解服务到非常小的粒度，造成一个服务需要再调用很多服务才能完成自己的任务，这是因为他们都想要对方的数据和功能。

面向服务的一个目标是重用，怎么保证尽可能的重用？使所有的服务尽可能的小？进而让他们在不同的场景可以被复用？看起来逻辑是对的，但问题是，每次我们这样复用服务时也增加了耦合。面向服务的另一个目标是减少耦合，减少耦合才能在保持商业和技术的要求的前提下很容易的对服务进行修改。

在 [SOA: synchronous communication, data ownership and coupling](http://www.tigerteam.dk/2014/soa-synchronous-communication-data-ownership-and-coupling/) 一文中，我们讨论了同步双向通信导致的一些复杂的耦合令人绝望：

- 通信相关的耦合（数据和服务不总是在一个服务内）
- 分层耦合（业务和安全、持久化等不在一个服务内）
- 时间耦合（当服务依赖的其他服务无法访问时服务不可用）

耦合有导致级联副作用的倾向。当修改服务协议时，所用使用该服务的其他服务都要跟着修改。当服务不可用时，所有依赖于该服务的其他服务也不可用了。当一个服务更新数据失败，所在 Process Services 的其他服务也需要应对更新失败的影响。

![错误是在服务收到消息执行更新操作前发生的还是更新操作后发生的？](https://github.com/hotjk/translation/blob/master/microservices/Image/Service-call-failing.png?raw=true)

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

![同步双向通信方式集成事务补偿](https://github.com/hotjk/translation/blob/master/microservices/Image/synchronous-SOA-orchestration.png?raw=true)

假设一种极端的微服务架构，每个服务只负责实体的一个属性（比如姓名、地址、邮编、城市等等），延迟时间会是一个巨大的问题，稳定性、协调服务调用和补偿也会是巨大的问题。我们一定是没找到正确的服务设计指导准则。

下一篇文章我们将谈到怎么在分布式环境集成服务，并看一下服务粒度对集成的影响。

------------

在本文第一部分，我们讨论了不能使用代码行数来确定服务大小是否合适，代码行数用来确定服务是否具备正确的职责也是完全无意义的。

我们还谈到了服务双向同步通信导致的强耦合问题和其他的讨厌的地方：

- 导致通信相关的耦合（因为数据和逻辑不总在一个服务内），也导致服务合同、数据、功能耦合以及网络通信的高延迟时间
- 分层耦合（持久化不总在一个服务内）
- 时间耦合（当服务依赖的其他服务无法访问时服务不可用）
- 服务依赖其他服务减少了自身的自治能力，也让服务更不可靠
- 所有这些导致在没有可靠消息和事务的情况下复杂的逻辑补偿

![可复用的服务，双向同步通信和耦合](https://github.com/hotjk/translation/blob/master/microservices/Image/reusable-services-and-coupling.png?raw=true)

如果我们组合同步双向通信的小服务也好，微服务也好，依据 1 class = 1 service 的建模方式，我们实际上倒退回了 90 年代的 Corba、J2EE、分布式对象的年代。
不幸的是，新一代的开发人员并没有分布式对象的使用经验，他们没参与过此类项目，也不了解这些想法是多么可怕，他们在重复这段历史，只是换了一些新的技术，比如用 HTTP 代替了 RMI 和 IIOP。

Jay Kreps 非常恰当的概括了目前微服务使用双向通信的方式：

![Jay Kreps - 微服务 == 潮人的分布式对象](https://github.com/hotjk/translation/blob/master/microservices/Image/Jay-Kreps-Microservices-equals-distributed-objects.png?raw=true)

仅仅因为微服务倾向于使用 HTTP、JSON、REST 并没有填补远程通信的劣势。新手常常忽略的分布式计算的劣势可以归纳为 8 个分布式计算谬论：

他们相信：

1. 网络可靠。不管怎样，任何一个碰到过服务器连接断开，或者因为路由器、交换机、WIFI 等问题导致 Internet 无法连接的人，或多或少都知道这是个谬论。
2. 延迟是零。网络调用相对于进程内调用的高额成本常常被忽视。带宽有限，相对于以前纳秒级调用，网络调用的延迟是毫秒级的。连续执行的调用越多，整体的延迟越大。
3. 带宽无限。实际上，即使 10G 带宽也比进程内内存调用慢的多，服务越小，调用发送的数据越多，服务越小，调用的次数越多，带宽影响越大。
4. 网络是安全的。国家安全局会告诉你为什么这是个谬论。
5. 拓扑不变。现实不是这样，产品环境的服务将经历运行环境的不断变化。旧的服务器会升级或替换，IP 地址也会变，防火墙等网络设备也会重新配置。
6. 一人管理。大规模安装配置需要多个管理员，网络管理员，Windows 管理员，Unix 管理员，数据库管理员等。
7. 传输成本为零。看一下内存对象和 JSON、XML 等各种格式之间序列化/反序列化的成本。
7. 同类网络。多数网络是由不同操作系统下支持各种网络协议、各种品牌的网络设备组成的。

以上八个分布式计算谬论还远没有完，如果你还有兴趣，Arnon Rotem-Gal-Oz 做了[更详细的回顾](http://www.rgoarchitects.com/Files/fallacies.pdf)。

## 有没有服务间双向同步调用的替换方案？

Pat Hellands 在 [Life Beyond Distributed Transactions – An Apostate’s Opinion](http://www-db.cs.wisc.edu/cidr/cidr2007/papers/cidr07p15.pdf) 一文中基本给出了答案。
文中 Pat 谈到理性的人不会使用分布式事务来协调事务边界。有很多理由不使用分布式事务：

- 事务启动时会锁定资源。服务应该是自治的，因此如果另一个服务通过一个分布式事务锁定了某个资源，就违反了服务自治规则。
- 不能期望服务在指定的时间内完成。服务是自治的，因此能自己决定怎样以及何时执行内部的处理。这意味着服务链中最弱的一环决定了服务链的强度。
- 锁定阻止其他事务完成他们的工作
- 锁定导致不能扩展。如果一个事务花费 200 毫秒锁定了一张表，那么服务最多每秒只能接受 5 个并发事务。增加再多的机器都不能改变每个事务锁定表格的时间。
- 2 阶段/3 阶段/ X 阶段提交的分布式事务是脆弱的设计。即使 X 阶段提交分布式事务以牺牲性能为代价解决了协调跨事务边界更新的问题。
仍然还有很多将 X 阶段提交留着未知状态的错误情景。（比如两阶段提交在提交阶段被中断，意味着一些参与者已经承诺了他们的修改，而另一些则没有。如果只有一个参与者失败了，提交将处于不可用的尴尬状态）

![两阶段提交协议的流程](https://github.com/hotjk/translation/blob/master/microservices/Image/2-phase-commit-protocol-flow.png?raw=true)

如果分布式事务不是解决方案，解决方案是什么？

解决方案分三步：

1. 怎么拆分数据和服务
2. 怎样发现数据和服务
3. 数据服务间如何通信

## 如何拆分和标记数据和服务

Pat Helland 认为，数据必须被集中成实体，实体需要限定大小以保证在事务范围内的一致性。
这就意味着实体不能突破单台机器的限制（跨机器的一致性保证需要分布式事务，这是我们之前希望避免的）。这也需要实体相对于更改实体的用例来说不能太小，如果实体太小，我们就又回到了需要使用分布式事务来保证跨服务更新的老路上。
经验法则：一个事务只处理一个实体。

来个真实世界的例子：
我之前一个项目可以做反面教材。该项目为了重用把服务分的过细，代价是服务稳定性、事务性、低耦合、低延迟都失去了。

客户想确保最大化的重用两个领域概念，LegalEntity 和 Address，任何 LegalEntity 需要使用地址的场合都要用到 Address，比如家庭地址、工作地址等等。为了协调创建、更新、读取以及确保重用，引入了一个 Task Service，"Legal Entity Task Service" 将会协调 "Legal Entity Micro Service" 和 "Address Micro Service"，我们选择让 "Legal Entity Task Service" 是承担 Task Service 的角色，但是这并没有解决我们要讨论的最重要的事务问题。
创建一个 LegalEntity，比如个人或者公司，我们首先通过 "Legal Entity Micro Service" 生成一个 LegalEntity，同时使用 "Address Micro Service" 生成了一个或多个 Address（"Legal Entity Task Service" 中的 CreateLegalEntity() 决定了创建几个 Address），每个 Address 都有从 "Address Micro Service" 的 CreateAddress() 方法返回的 AddressId，"Legal Entity Micro Service" 的 CreateLogalEntity() 方法通过 "Legal Entity Micro Service" 里的 AssociateLegalEntityWithAddress() 方法将 LegalEntityId 和 AddressId 关联起来。

![不正确的微服务](https://github.com/hotjk/translation/blob/master/microservices/Image/Bad-microservices-Create.png?raw=true)

从上面的序列图，我们清楚的看到了各种级别的深度的耦合，如果 "Address Micro Service" 没有应答，就不能创建 LegalEntity，这种方案的延迟很高，因为有太多的远程调用，使用并行调用可以减少延迟，但是这种小的优化解决不了问题的本质，事务问题仍然存在。

假设 CreateAddress() 或者 AssociateLegalEntityWithAddress() 中的一个调用失败，我们就留下了一条脏记录，我们创建了一个 LegalEntity 但是其中一个 CreateAddress() 调用失败，导致系统数据不一致性，因为并不是所有的数据都创建成功。即使我们成功的创建了 LegalEntity 以及相关的全部地址，也会出现某些时候获取 LegalEntity 时无法完整获得该实体全部 Address 的情况，这也造成系统的不一致性。

这种服务的组合调用造成了 "Legal Entity Micro Service" 里的 CreateLegalEntity() 方法不堪重负，现在他得负责在调用失败时重试以及产生的一系列清理工作（补偿）。当某个服务的清理工作失败，你想怎么做？"Legal Entity Micro Service" 里的 CreateLegalEntity() 在尝试重试一个失败的服务调用或者执行清理工作时发生服务器物理故障怎么办？开发人员能保证 CreateLegalEntity() 方法的正确实现么？ 开发者能确保 CreateAddress() 方法和 AssociateLegalEntityWithAddress() 方法是幂等的？如果不能，重试服务调用时会不会创建两个 Address 对象或者把 Address 跟 LegalEntity 关联了两次？

## 事务性问题可以通过研究用例进而对重用粒度重新评估来解决

LegalEntity 和 Address 服务的设计需要架构团队设计一个合乎逻辑的标准模型，并从中发掘出什么是可重用的服务。这种方式的问题是，一个标准数据模型没有考虑数据是如何使用的，也就是没有考虑该数据用例。究其原因，数据修改/创建的方式决定了事务的边界，也就是我们说的一致性界限。根据经验，在一个事务或者说一个用例内会发生修改的数据，应该紧密相关且有统一的所有权。

所以我们的经验法则扩展为：1 usercase = 1 transaction = 1 entity

把地址提升为服务允许直接访问是我们犯的第二个错误。可以说，把重心放在重用带来了一种必须把地址包装成服务的感觉，这是在假设每个人都需要地址服务并且随时希望修改城市名称或者邮政编码，也许会有合理的原因将一些信息集中提供服务，但是每件事儿都做成这样成本就太高了。

数据模型看起来像这样：

![不好的微服务数据模型](https://github.com/hotjk/translation/blob/master/microservices/Image/Bad-microservices-data-model.png?raw=true)

如图中模型，LegalEntity 和 Address 的对应关系是共享直接关联，这就表示两个 LegalEntity 可以共享一个 Address 实例，实际业务中这种情况不会发生，因为两者是一种直接组合关联，更像一种父子关系。如果 LegalEntity 对象 被删除，Address 对象也就没有存储的必要了，父子关系表达了 LegalEntity 和 Address 密切的属于一个整体，他们被共同创造，一起改变，一起被使用。
这意味着我们不应该有两个实体，实际上只有一个实体，LegalEntity，一个或多个 Address 对象紧密的与实体关联。Pat Hellands 的实体一词是从领域驱动设计（DDD）中得来，DDD 还包括更丰富的词汇：

- 实体（Entity）-- 描述一个可以用标识而不是它的数据来辨识的对象，比如 Legal Entity（它的子类 Person 用社会安全号来标识，子类 Company 用增值税税号来标识，等等）
- 值对象（Value Object）-- 描述一个可以用数据而不是标识来辨识的对象，比如地址、姓名、Email地址，两个同样类型的值对象，具备同样的值就可以认为两者相等。值对象不会独立存在，它总是作为一部分被关联在实体上，值对象作为实体的数据而存在。
- 聚合（Aggregate）-- 一群有着复杂关系的对象串在一起，聚合用于保证其关联对象的一致性。聚合也用于锁控制，以及确保事务一致性。
  - 聚合选择一个实体作为根并要求只能通过聚合根访问聚合内的对象。
  - 聚合通过 ID 来唯一标识（常常是 UUID 后者 GUID）。
  - 聚合之间通过 ID 来引用，聚合之间绝对不要用内存指针或者关联表来引用。

从描述中我们能断定 Pat Helland 说到的实体在 DDD 的官方词汇里叫做聚合（Aggregate），DDD 的词汇更丰富，所以以后我们将继续使用 DDD 的词汇，如果你对聚合感兴趣，推荐阅读 Gojko Adzic 的[文章](http://gojko.net/2009/06/23/improving-performance-and-scalability-with-ddd/)。

通过对用例的分析（LegalEntity 和 Address 是同时创建和修改），再加上 DDD 的词汇（LegalEntity 是实体也是聚合根，Address 是值对象），现在可以重新设计数据模型（也被称为领域模型）。

![LegalEntity 微服务 -- 更好的模型](https://github.com/hotjk/translation/blob/master/microservices/Image/LegalEntity-Microservice-better-model.png?raw=true)

上图的设计中，Address 中不再包含 AddressId，因为值对象不需要 ID 来标识。
LegalEntity 仍然有 LegalEntityId，并且必须通过 LegalEntityId 来使用 LegalEntity 微服务。
在新的设计中，Address 服务已经废弃，只留下 "Legal Entity Micro Service"。

![更好的 LegalEntity 微服务](https://github.com/hotjk/translation/blob/master/microservices/Image/LegalEntity-microservice.png?raw=true)

新的设计完全解决了事务问题，因为例子中只有一个服务。还有很多可以改进的，我们还没有谈到服务间如何通信，以及处理过程或用例要跨越多个聚合或服务时，如何确保整个服务的协调性和一致性，

------------

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

![服务复用在双向同步通信方式下的耦合](https://github.com/hotjk/translation/blob/master/microservices/Image/reusable-services-and-coupling.png?raw=true)

### 如果同步通信不是办法，那么是不是应该用异步通信？

是的，但是取决于…… 说取决于什么之前，我们先看看同步和异步通信的特点：

- 同步通信需要消费者和提供者通信发生时都在线，比如载入一个网页或者两个人对话。
- 异步通信的发送者和接收者不需要同时在线，异步通信方式打破了发送者和接收者之间的临时耦合，比如电子邮件、短信、邮寄。

基于这些特点，我们简单的对通信进行了分类：

### 同步通信是双向通信

![同步通信](https://github.com/hotjk/translation/blob/master/microservices/Image/synchronous-communication.png?raw=true)

上图的同步通信方式称为 Request/Response 模式，典型情况下以 PRC 模式实现。
在 Request/Response 模式下，服务消费者发送一个请求消息给服务提供者，在服务提供者处理请求消息时，服务消费者基本上只能等待直到收到应答或者发生错误（有些人可能会说，消费者可以利用异步平台特性，在等待时并行的做其他调用，但这其实没有解决调用时消费者和提供者的时间耦合，在收到应答前消费者根本无法继续后面的工作）。典型的 Request/Response 或者 RPC 调用流程如下图：

![同步通信流程](https://github.com/hotjk/translation/blob/master/microservices/Image/Sync-waits.png?raw=true)

如图所示，服务消费者和服务提供者之间是强耦合的，服务提供者不可用时，服务消费者也不能工作，这类时间耦合或者说运行时耦合是我们应该尽量避免的。

### 异步通信是单向通信

![异步通信](https://github.com/hotjk/translation/blob/master/microservices/Image/asynchronous-communication.png?raw=true)

使用异步通信时，发送者通过传输通道发出一个消息给接收者，发送者只需要短暂的等待就可以获得消息送达的反馈，然后发送者就可以继续后面的工作。这就是单向通信的本质，典型的执行路径如下图：

![异步通信流程](https://github.com/hotjk/translation/blob/master/microservices/Image/async-waits.png?raw=true)

异步通信就是大家常说的消息传送，异步通信的传输通道负责接受发送者发来的消息，并负责把消息交付给接收者（可能有多个）。可以说传输通道承担了消息交换的职责，传输通道可以非常简单（比如使用 Socket、0MQ），也可以使用带有持久化的高级分布式解决方案（比如 ActiveMQ、HornetQ、kafka 等等）。消息机制和异步通信通道提供了不同消息交换保障：交付保证和消息顺序保证。

### 为什么异步通信不是完美的解决方案？

事实上，很遗憾，异步通信不像同步通信一样容易，服务的集成模式决定了真实的耦合程度。

双向通信有几种形式或者变化：

- 远程过程调用（RPC）-- 异步通信
- Request/Response  -- 同步通信
- Request/Reply  -- 也被称为基于异步的同步通信。

经常看到一些项目使用 Request/Reply （基于异步的同步通信）来确保服务不会产生时间耦合。说到细节就比较讨厌了，从消费者来看，大多数使用 Request/Reply 模式的应用程序，时间耦合的程度跟使用 RPC 或者 Request/Response 没有太大的区别，他们统统是双向通信的变种，都要求发送者等到应答后才能继续处理业务。

![RPC 和 Request/Response -- 同步双向通信 VS. Request/Reply -- 异步双向通信](https://github.com/hotjk/translation/blob/master/microservices/Image/RPC-Request-Response-vs-Request-Reply.png?raw=true)

### 那么结论是？

结论就是服务间双向通信是问题的根源，我们把服务粒度变小（变成微服务），这个问题也不会变小。
我们已经看到异步通信可以打破服务间的时间耦合，但是只有这种异步通信是单向时才真正起作用。

![我们的方案:)](https://github.com/hotjk/translation/blob/master/microservices/Image/asynchronous-communication.png?raw=true)

问题是怎样在只使用服务间异步单向通信的限制下设计你的服务或微服务（UI 和服务之间通信时另一件事，我们后面会谈到）？

下一篇我们看看如何拆分服务以及怎样让通过异步单行通信调用它们。

------------

第三部分我们谈到，为了确保我们的服务能高度自治，我们需要避免服务间的（同步）双向通信，而应该使用单向通信。

更高级别的自治同时也意味着更少的耦合，耦合越少，服务和数据的版本问题也就越少。
我们还提高了服务的稳定性 -- 其他服务执行时发生的错误不会直接影响我们的服务。

但是，怎样才能完成工作，如果我们只使用单向通信，我们的服务怎么返回数据呢？
直接的回答是你不能，但是拥有良好定义的服务边界，你的服务就不需要直接调用其它服务来获取数据，大多数情况是这样。

## 服务边界

什么是服务边界？这个词基本上表示一个服务负责的业务数据和业务功能。在 [SOA: synchronous communication, data ownership and coupling](http://www.tigerteam.dk/2014/soa-synchronous-communication-data-ownership-and-coupling/) 一文中对服务原则，比如边界和自治有更详细的介绍。
边界确定了那些是服务的内部，那些是服务的外部，在本文第二部分我们使用了聚合模式分析了那些数据属于 Legal Entity Service 的内部。
在 Legal Entity Service 的例子中，我们意识到 LegalEntity 和 Address 的关系是从属关系，因为 LegalEntity 和它相关的 Address 是同时创建、修改和删除的。把两个服务合并成一个服务，Legal Entity Service 做到了自治，进而避免了调用服务时出现错误时，要对其他操作数据服务的调用做编排的复杂场景。

Legal Entity 耦合的问题可以被轻松解决，但是如果数据和数据见的关系更复杂怎么办？仅仅把这些数据堆到一个服务里避免跨处理边界的数据变更问题（比如不同的服务部署在不同的操作系统进程或者不同的物理服务器上），这样做的话很快我们就会被拉回 Monolith 的方式。Monolith 本身没什么错，也可以使用许多相似的设计原则来构建 Monolith 应用，比如可以用模块来代替微服务，模块也可以一起打包后一起部署，就像微服务经常被独立部署一样（独立部署常常被认为是微服务带来的优点）。

## 模糊的边界 -- Monolith 滑坡

Monolith 的一个问题是模糊边界的风险。因为模块是紧密绑定在一起的，常常在同一个代码库内，这样会造成一种缓慢恶化的倾向，导致了模块间越来越多的耦合，因为实在是太容易调用其它函数、组件或者 Join 其他表。

Monolith 感觉好极了，特别是在项目初期，问题很少，复杂度也不高，Monolith 还有一种趋势，会导致其以数据和功能逻辑的形式承担过多的职责。

开发 Monolith 应用时，你可以：

- 本地优势。比如执行内存调用避免分布式事务，Join 处于同一个 DB 下的其他 SQL 表格
- 利用开发工具的重构、代码自动补全、代码查询等能力

硬币的另一面是高耦合和低内聚的风险，Monolith 很容易慢慢的变大，因为他承担了太多的职责，因为太容易使用已有的功能和数据。

![Monolith 数据模型慢慢变大最终因为缺乏内聚而变得混乱](https://github.com/hotjk/translation/blob/master/microservices/Image/model-growth.png?raw=true)

这就是 Monolith 走下坡路：

![Monolith 随着复杂度下滑](https://github.com/hotjk/translation/blob/master/microservices/Image/monolith-slippery-slope.png?raw=true)

Monolith 还有几个劣势：

- 很难适应新技术 -- 要使用新的框架/语言/技术常常需要重写整个 Monolith 应用
- 可重用性差 -- 功能部分不能单独重用
- 交付缓慢 -- 新功能引入需要协调其他功能一起发布
- 应用的大小和职责越来越大
- 越来越高的耦合
- 启动要用很长时间
- 测试要用很长时间
- Monolith 要求你的大脑要同时容纳大量的业务概念，而人对复杂度的掌控是有限的
- 可靠性 -- 一个业务出了问题（比如内存溢出）会导致整个 Monolith 系统宕机

你可以在设计 Monolith 时定义内部的服务和组件并且定义好边界使他们保持松散的耦合，但是从我 20 多年的经验来看，使用这种模式的应用基本都是大泥球。

## 集成成堆的 Web Service

我看到的许多机构实施 SOA 的方案都是在已存在的 Monolith 系统上加上 Web Service，这种方式有其意义，可以使老的 Monolith 系统被复用。

问题是大多数 Monolith 系统已经发展到了包含许多不同类型的业务。这意味着公司最终不得不有多个主系统，这些系统拥有很多相似的业务数据，每种业务都没有真正的单一源头。

![Monolith 集成成堆的 Web Service](https://github.com/hotjk/translation/blob/master/microservices/Image/integration-by-a-bunch-of-webservices.png?raw=true)

如果我们刚接手一个 Monolith 系统，想要拆分成小一点的服务，我们还需要处理 Monolith 系统的内部耦合，典型的内部耦合包括集成、直接的方法调用、SQL Join 等等。如果我们只是简单的创建服务只会让事情更糟。

![Monolith 分拆成服务](https://github.com/hotjk/translation/blob/master/microservices/Image/monolith-sliced-up-into-microservices.png?raw=true)

所有这些都导致了服务边界模糊，一些服务会很单薄并且会过度的依赖其他服务的数据和功能。在我看来，这不是松耦合，也许恰恰相反。

## 定义服务边界

当创建新服务或者从现有的 Monolith 分拆服务时，我们需要花时间定义服务的边界，所以我们可以避免使用服务间双向通信。
注意：高度自治并不是所有情况都需要的，可能有些场景使用双向通信时从开发角度看性价比更高，有些场景下缺乏自治是可以容忍的（比如跨越多个服务的读取）。

在旧有的 Monolith 系统，我们可能已经收集了关于零售领域的所有的功能和数据，可能包含的功能区有，商品目录、销售、库存、配送、结账，每一个功能区可以被叫做子域或者业务功能：

![零售领域的功能区](https://github.com/hotjk/translation/blob/master/microservices/Image/functional-areas.png?raw=true)

零售业务就是销售商品，所以每个功能区或者说子域都会以一些方式牵扯到商品（Product）这个领域概念：

- 商品目录子域里面的商品（Product）包含名称、描述、图片等等。
- 销售子域会创建商品订单。
- 库存子域关心商品的库存数量（QOH），还有商品所在位置等，这里我们用 Product 这个名字，也可以不用，根据库存相关的项目/商品的状态，有时候也可以叫 Stock Item 或者 Back Ordered Item 等等。
- 价格子域关心商品的定价策略，可以包含根据客户状态的折扣（客户状态可能在另一个 CRM 大系统或服务里维护）。
- 配送子域关心商品的尺寸、重量以及发送目的地等。

如你所想，所有子域都跟商品发生了关联，子域可以给商品使用 Product 这个名字也可以给商品使用其他的名字（其他的领域概念比如 Customer 的命名也类似）。关联在商品上不同的数据也很有意思，库存子域关心库存单元（SKU）、库存数量（QOH）、库存位置代码，对库存子域来说，商品名称、图片之类的可能不关心，即使有了，也只是给库存工人工作提供帮助，不会是处理库存业务必须的。
另一方面，配送子域不会关心库存数量（QOH）、库存位置代码等等，他们关心商品尺寸、重量，也许会关心商品名称，如果要打印配送收据的话。
配送领域的这种不同视角在领域驱动设计（DDD）里被称为不同的界限上下文（Bounded Context）。

在 Monolith 系统，很容易会创建一个 Product 表，包含许多的属性和关联，不同子域只是在他们认为合适的地方插入/更新或者关联数据，风险是 Product 领域模型会变大，造成该模型变化的业务会有很多（违反单一职责原则），导致耦合和一致性缺失。
因为其他业务依赖它，你不能修改 Product 表结构，从托管代码到服务和服务协议的提升只是移除了技术耦合，实际上服务仍然需要其他服务的数据和功能，这将我们服务的自治性降低到不可接受的级别。

### 定义服务边界

我们需要一种方法来设计服务边界，这样我们的服务就可以不需要为获取数据或者调用功能而使用双向通信方式来互相沟通。

我们可以围绕着功能区或者业务范围来开始服务构建，并用这个作为服务的边界，服务拥有自己的数据和功能。
别的服务不允许拥有这个服务的的数据。
数据只能有一个所有者，有了这个保证，我们相信我们的服务是他所有的业务数据的唯一事实来源。

这样做确保我们的服务只需要响应业务功能负责的修改，这就是服务的单一职责原则（SRP），你可以越多关于 SRP 的谈论[这篇](http://www.udidahan.com/2014/05/26/people-politics-and-the-single-responsibility-principle/)和[这篇](http://blog.8thlight.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html)。

注意：下面的例子是构建更松散耦合服务之路的第一步，定义服务边界可不容易，后面的文章会继续深入讨论如何用比我们现在描述的基本方法更好的方式来定义一致服务边界。我想很多人会争论商品目录是不是一个最好的服务边界，但是既然很多企业都用了商品目录的概念，这里我们也就沿用这个概念。

让我们开始商品目录服务的设计，商品聚合包含很多属性，比如名称、ID（聚合需要唯一 ID）、图片、描述等等，商品目录服务会是商品聚合的唯一事实来源。在销售服务中我们关心用户通过网上商城等途径选择商品构建订单，在销售服务我们关心每个具体的订单行特定 ID 商品对应的数量，在销售服务构建订单对象时我们不需要商品的名称。（网上商城需要商品的名称，但是这个用例只是读商品名称，在构建销售服务的订单对象时我们关注的是写用例）

服务模型和边界看起来是这样（简化版）

![简化的服务数据模型](https://github.com/hotjk/translation/blob/master/microservices/Image/simple-service-data-models.png?raw=true)

上面的服务领域模型展示了两个高内聚、低耦合、简单清晰的数据模型。两者唯一的耦合是订单行通过商品 ID 引用了商品（记得本文第二部分说过聚合间通过彼此 ID 引用）。
网上商城（消费许多服务的客户端）负责显示销售的商品，客户购买商品的价格，当用户塞满购物筐，网上商城会发送包含用户想购买的商品的数量、价格、商品 ID 的命令消息给销售服务。后面我们会看到我们怎样使用复合式 UI 的优势来降低网上商城的耦合，现在我们只是假设网上商城这个客户端使用双向通信方式逐个调用服务。
只要提供了数量、单价、和商品 ID 给销售服务，它就能创建订单、添加订单行，而不需要联系商品目录服务。

但是销售服务想要获取客户的订单确认时怎们办？当用户通过电子邮件收到订单确认，客户不单要看到价格、单价和商品 ID，客户还需要知道购买的商品的名称，甚至图片，这样客户才能确认订单里是他希望购买的商品。

在准备订单确认邮件时，销售服务需要保留从商品目录服务获得的商品名称么？
让我们看一下销售服务可能的做法：

- 常规方法：销售服务使用双向通信方式针对订单里的每个订单行调用商品目录服务（可以逐个调用，也可把所有订单行对应的商品打包一次调用获得一个集合）
  - 这意味着销售服务现在跟商品目录服务有更强的接口协议耦合和时间耦合，销售服务需要知道商品目录服务提供的具体的操作和数据。
  - 这意味着任何商品目录服务不向后兼容的改动都会连带造成销售服务的修改（即使这些改动是销售服务不关心的），或者商品目录服务需要对接口协议版本化。
  - 问题可以一定程度的解决，如果商品目录服务提供消费者驱动的接口协议，针对特定的客户，比如销售服务，提供客户特定的接口协议。
  - 如果商品目录服务宕机，因为时间耦合，销售服务不能进行订单确认了，这也许不是个大问题，因为订单确认的时间并不是特别重要，况且订单确认也不直接与客户交互。
  - 如果销售服务也负责渲染网上商城包含商品的界面，那么销售服务和商品目录服务的时间耦合就很严重了，仅仅因为商品目录不可用（比如因为 ERP 系统故障）就导致销售服务不能创建和接受新订单了。
- 2015年2月28日补充：还有其他有别于自治服务的 SOA 方案，其中一个值得一提，在[这个方案](http://www.infoq.com/minibooks/composite-software-construction)中，服务不需要自治也不需要拥有业务数据，不需要暴露刻意的接口，只需要负责协调不同记录系统（SoR）之间的交互。商品分类服务和销售服务将被归入记录系统（SoR），并且仍然是自治的，一个新的负责 “发送订单确认邮件” 用例的协调服务被引入进来，这个服务将会调用商品目录 SoR 和销售 SoR 来获取订单信息和商品信息，并将信息组合起来完成订单确认。服务操作可以仍然使用事件触发。
- 商品目录服务拆解到订单确认过程
  - 这种耦合更轻量更精细，因为销售服务并不需要知道商品目录服务内部数据和协议（除了 UI -定义的一个非常小的共享渲染上下文）。
  - 业务混搭仍然涉及到服务之间的时间耦合。
  - 我将在后面的文章中再谈组合 UI 混搭和服务混搭。
- 最后的方案是销售服务包含商品目录数据的缓存或者复制，这可以工作，没有对商品目录服务的时间耦合，通过使用基于事件的数据复制。

### 通过事件进行数据复制

当从商品目录添加、修改或者删除商品时，我们注意到可以通过业务事件通知其他服务这一事实。
这种情况下，从业务角度来看商品分类服务非常简单，业务事件类似大家熟悉的增删改事件：ProductAdded、ProductUpdated
、ProductDeleted。
注意，事件以过去式来命名，事实上这很重要。

我们可以让销售服务通过消息通道（比如发布/订阅风格）监听这些事件，并允许销售服务建立其内部的商品对象，并将自己关心的数据放入新的商品对象：

![通过事件进行数据复制](https://github.com/hotjk/translation/blob/master/microservices/Image/data-duplication-over-events.png?raw=true)

对应的服务数据模型：

![包含时间和数据复制的服务模型](https://github.com/hotjk/translation/blob/master/microservices/Image/Data-duplication.png?raw=true)

使用数据复制方案获得以下优势：

- 数据所有权清晰，商品目录是数据的所有者，当数据变更时通知相关服务。
   - 这种形式的数据缓存技术比多数传统缓存机制更好。数据变更时立即可以获得事件通知，传统的缓存机制无法从数据所有者那里获得即时的通知和指示，这会导致自己保留了无效的数据。
- 协议耦合度低。你只会绑定到仅包含数据的事件协议，事件协议比既包含数据也包含功能的传统 WSDL 服务协议简单的多。经验表明，事件协议往往比包含功能的标准协议更稳定。不过设计时仍然必须考虑向前兼容，才可能在添加非强制字段时不导致现有的事件订阅者出问题。
- 商品目录服务和销售服务之间的耦合度非常低。
  - 销售服务只需要知道事件协议和消息通道地址。
  - 商品目录服务没有任何对销售服务的耦合，他不知道销售服务收到事件后会怎么处理，事实上商品目录都不知道什么服务收到了它发出的事件。
- 以牺牲强一致性保留最终一致性为代价解除了时间耦合和技术耦合。
  - Pat Hellands 的 [Life Beyond Distributed Transactions – An Apostate’s Opinion](http://www-db.cs.wisc.edu/cidr/cidr2007/papers/cidr07p15.pdf) 一文中有值得学习的地方。文中，他总结了你只需要在一个聚合实例内保持一致性（也就是在一个服务的一个事务内），聚合实例间只需要最终一致性，因为我们没有办法在聚合实例之间保持强一致性，除非我们打算付出分布式事务的高额代价。
  - 在这里，最终一致性意味着如果消息通道故障导致消息不能发送给销售服务，即使商品名称变更了，在订单确认时我们使用的还是原来的商品名称，一旦消息通道恢复可用，销售服务会收到商品目录服务发送来的消息，当使用缓存时，只能遵循最终一致性的标准，不管是否使用事件机制。
  - 通过把事件跟时间关联，我们可以减少最终一致性造成的问题。事件包含名称和数据，在数据中你可以通知接收者未来多长时间内数据中的值是有效且可以被缓存的（比如价格只会每天变化，商品名称很少变化，等等）。

怀疑论者可以觉得通过事件做数据复制比起使用现有的数据库技术来实现要多做很多事，这个观念倒也没什么错，通过事件进行数据复制也有它的复杂性，比如监控、通道配置、IO 成本，因为复制了数据，服务还需要更多的内存和存储空间。
通过事件进行数据复制是一种众所周知、技术中立的模式用于慢慢的拆分 Monolith 应用到自治服务，但是基于事件的集成方案不是最终解决方案。
当我们走的更远，就会有更多收获，这个收获就是，事件可以驱动业务处理。

### 使用跨服务业务事件驱动业务处理

如果我们把增删改事件提升为真实的业务事件，以反应聚合的状态变化，那么我们就可以用这些事件驱动业务操作，代替调度中心（编排）通过双向通信方式来协调业务处理。

让我们看看怎样使用事件来驱动订单处理流程。客户在网上商城按下接受按钮时，触发一个 AcceptOrder 命令消息发送（通常是异步）给销售服务。

![OrderAccepted 事件驱动订单处理流程](https://github.com/hotjk/translation/blob/master/microservices/Image/OrderAccepted1.png?raw=true)

AccpetOrder 命令导致订单状态改变，订单状态被转换为 Accepted 状态。
这种状态改变通过 OrderAccpeted 事件传达给所有对此感兴趣的服务，我们在描述一个订单已经被接受，这件事是不可撤销的（我们可以补偿，但是不能回滚这次改变）。
销售服务不知道谁对这个事件感兴趣，但是从公司业务层面，我们已经梳理了订单处理过程并决定了那些服务需要对 OrderAccepted 事件做出反馈。
这被称为响应式编程或者事件驱动架构（EDA），这跟传统的使用 “业务流程执行语言” 编排服务并指导服务做什么的方案很不一样。

EDA 服务自己决定当事件发生时要做什么，当我们需要协调多个服务的场合，比如确保客户已经付款并且订单里的所有商品都有货的时候才可以发货，我们将引入一个新的聚合来负责订单处理过程和能力。这个聚合是属于发货服务，还是一个独立的服务，现在还不重要，重要的是我们已经标识出一个明确可以分配职责的核心业务能力。

这样一个过程处理聚合可以实现为一个 Process Manager 或者 Saga（在 Rebus 和 NServiceBus 里这么叫），如果需要的话，Process Manager 可以选择指导其他服务具体做什么（也就是局部编排），但是通常通过单独使用事件就能解决。

在下面的例子中，订单处理服务发送 OrderReadyForShipping 事件之前等待两个事件：OrderAccepted 和 CustomerBilled，两个事件需要包含足够的信息，表达出他们是同一个订单处理过程，可以通过 OrderID，也可以通过其他形式的相关 ID，这种服务间使用事件的协作方式也被称作编排，可以被看成对更传统的编排的补充（真实的解决方案会联合使用两种编排方式）。

![编排的订单处理过程](https://github.com/hotjk/translation/blob/master/microservices/Image/orderfulfilment-process.png?raw=true)

事件驱动架构已经说的够多了，这篇已经太长了，服务边界定义放到下次吧。

------------

第四部分我们建议围绕功能区或者业务功能/界限上下文来构建服务。我们讨论了附属于业务能力的业务数据和逻辑必须被封装在服务里以便确保数据的单一源头。这也意味着其他服务不允许拥有别的服务已经拥有的数据，因为我们想要服务自治（也就是可以不需要和其他服务同步通信就可以工作）。我们也看了如何避免服务间双向通信（RPC，REST 或者 请求/应答），我们找到的方案是组合 UI 以及通过事件进行数据复制。我们也简单讨论了第三种方案：这个方案引入了服务的不同视角，这种方案下服务不需要自治，服务不暴露刻意的接口，而是协调多个自治的纪录系统（SoR）的更新和读取，许多有大遗留系统的公司应该尝试使用第三种方案，比起开发新的对应业务能力的自治服务来说摩擦更少。

第五部分继续讨论根据自治的服务构建 SOA 和微服务。

## 业务能力和服务

第四部分我们建议围绕功能区或者业务功能/界限上下文来构建服务。
我想语气应该更强硬一点：我们应该让服务匹配业务功能。

为什么？Bill Poole 在 [Business Capabilities](http://bill-poole.blogspot.dk/2008/07/business-capabilities.html) 文中解释了为什么服务匹配业务功能是正确的方式：

> “业务功能是什么？对机构来说，业务功能是指那些在某些方面上对机构的整体业务做出的贡献。

> 业务功能的优点是其非常的稳定，如果我们看一个典型的保险机构，它应该有销售、市场、保单管理、索赔管理、风险评估、开票、支付、客户服务、人力资源管理、渠道管理、费用管理、合规性、IT 支持和任务管理等业务，实际上，任何一个保险机构都会需要这些业务” -- Bill Poole

业务功能是我们开发的软件的根基，Dan North 说过下面这段话：

![Dan North：业务功能是资产](https://github.com/hotjk/translation/blob/master/microservices/Image/dan-north-on-business-capabilities.png?raw=true)

最后 Udi Dahan 在 [The Known Unknowns of SOA](http://www.udidahan.com/2010/11/15/the-known-unknowns-of-soa/) 一文中说明了为什么他觉得服务应该是自治的，并成为特定业务功能的技术权威。

> “同步的生产者/消费者意味着模型服务不调用其它服务就不能满足他们的目标。为了让我们能够实现 SOA 承诺的 IT/业务一致性，我们需要服务是自治的，也就是能够不依赖那种外部分就能完成自己的目标。

> 服务是特定业务功能的技术权威，任何数据和业务规则都必须只属于一个服务。

> 这意味着，即使服务发布和订阅对方的事件，我们总是知道每一个数据块和规则的权威事实来源”  -- Udi Dahan

我把上面的描述总结为以下规则：

## 服务

- 是对于一个给定的业务功能的技术权威
- 是所有支持该业务功能的数据和业务规则的所有者 - 每个地方都是
- 形成了功能的单一事实源头
- 形成了业务和 IT 匹配，确保维持业务的自治性和封装性

推论是：部署后的服务需要在任何需要它的数据和逻辑功能的场合可用。

思考后会觉得更合理，Udi Dahan 在 [The Known Unknowns of SOA](http://udidahan.com/2010/11/15/the-known-unknowns-of-soa/) 一文中解释了原因：

> “从业务功能的视角来看一个服务，我们看到许多用户界面展示了属于不同的功能的信息，产品价格旁边显示了是否有库存，为了让我们能够满足服务的上述要求，这引导我们将用户界面理解为一个混搭，每个服务通过它特有的数据处理 UI 中的一个区块。

> 最后，用 Web 应用程序、后端、批处理这样进程边界作为服务的边界是非常差的，我们希望多个业务能力承载于多个进程中。” -- Udi Dahan

太抽象了，我么举个复合 UI 的例子：

![亚马逊的复合 UI](https://github.com/hotjk/translation/blob/master/microservices/Image/Composite-UI-Amazon.png?raw=true)

组合 UI 区块的工作可以在客户端完成也可以在服务端完成。这是思考方式是每个服务渲染它负责的 UI 成为网上商城页面上一个特定的 DIV：

![HTML 组合](https://github.com/hotjk/translation/blob/master/microservices/Image/HTML-Composition.png?raw=true)

更新：如果组合发生在客户端，那么客户端 UI 组件和服务端对应的部分（比如一个 REST 接口）通常包含双向通信（很少使用发布/订阅模式，除非你希望服务器生成事件推到客户端）。为了让 UI 有应答，典型情况下客户端 UI 组件和服务端对应的部分使用基于异步的同步双向通信，比如使用带有 Javascript 回调的异步 Ajax 调用。

组合 UI 的优点是，像网上商城这样的应用程序，不需要知道提供页面上 UI 区块的每个服务的任何细节，事实上，评论就是分数、评论数和赞的数量在评论服务里的完整封装，评论分数渲染成星星而不是数字是一个具体的应用程序可视化的决定，所有的评论服务输出可能是 &lt;score&gt;4.2&lt;/score&gt;，UI 按样式的要求渲染，从应用程序的角度来看，他们共享的是页面上下文和样式协议，事实上，评论服务的 UI 会被渲染成一个 id 为 “Book:Review” 的 DIV。
评论功能增加新功能只需要在评论服务里实现，不同的应用程序平台的 UI 区块需要针对新去求更新并推到应用程序，但是应用程序内部不需要修改什么，修改完全在评论服务内部。

使用复合 UI 的另一个优点是别的服务很少需要订阅其他服务的事件来建立缓存或数据副本，这个问题已经在组合层面解决了。

缺点是给定的服务需要能够提供它的 UI 区块给所有可能的应用程序，比如 iOS 应用、后台 .Net 应用，基于 Java 的网上商城等等，下面的使用组合方式的例子中多个服务联合渲染/打印出发票：

![复合 UI 发票例子](https://github.com/hotjk/translation/blob/master/microservices/Image/Composite-Invoice.png?raw=true)

我觉得，复合 UI 是自治服务真正强大的场合，从多个服务渲染内容却不会让服务之间耦合。你获得在各个层级封装良好的服务，我们展示出来的接口/协议很少暴露服务的数据和功能，服务通常暴露给服务的消费者，驱动本地 IT 操作接口（第三方/遗留系统集成）、事件（典型情况下只需要包含发生了什么事以及变更影响的聚合 ID）、几个对外暴露的命令，当然还有我们的 UI 区块，除了这些只有非常少的协议/接口暴露给生态系统。

更新：复合 UI 的一个挑战是当发生更新时（比如提交表单跨越多个服务），因为我们这里又遇到跨事务边界更新数据的问题，当一个或多个服务更新失败而其他成功时没有两阶段提交事务来帮助解决问题。你还必须考虑到浏览器是一个不可信赖的平台，由于浏览器挂起或者用户关闭了页面，就像服务端实现相似的编排。

如果你对复合 UI 感兴趣，可以阅读 Udi Dahan 的 [Service-Oriented Composition](http://www.udidahan.com/2012/06/23/ui-composition-techniques-for-correct-service-boundaries/, http://www.udidahan.com/2014/07/30/service-oriented-composition-with-video/) 一文。

## 服务部署模型

希望服务被部署后任何需要他的数据的地方都能使用，这其实是有问题的，因为服务是有物理结构或者边界的。

根据 [Philippe Kruchten's 4+1 view](http://en.wikipedia.org/wiki/4+1_architectural_view_model) 对架构的描述，逻辑视图、物理视图（部署视图）应该彼此独立（也就是说它们不应该一对一的映射）。

结合上面的服务定义，我们得到的结论是：服务必须以业务逻辑作为构造单元和边界。

总结为以下定义：

- 系统和应用是运行时进程边界，进程边界是物理边界（简单的例子是一个单一 .exe 或者 .war 部署单元）
- 服务是逻辑边界，不是物理边界，你可以选择把服务部署为一个单一的物理进程，但这也只是部署服务的一种方式，并且不一定是正确的方式。
- 因此进程/应用/系统边界和服务边界可能是不对应的

为了支持将多个服务组合成应用程序，每个服务可以有以下的部署选项：

- 许多服务可以被部署在同一个系统、应用程序或进程内
- 一个服务的不同部分可以被分别部署为不同平台（Java、.NET、iOS、Android、Web 等等）的应用程序，比如服务的 UI 部分可以被部署成一个 Web 应用程序，一个 iOS 应用程序，一个 Android 应用程序（每一个都是单独实现并针对特定平台打包，但是仍然属于同一个逻辑服务），举个例子：产品的价格在网上商城、后台应用、iOS 商业应用上都会显示（换句话说，业务功能跨越应用程序边界）
- 服务部署不需要强制分层，同样的服务可以部署到多层，甚至多个应用程序中。服务 A 和 B 的一部分可以被部署为同一个 Web 应用，另一部分可以被部署为同一个应用程序的后台或同一个应用程序服务。
- 许多服务可以被部署在同一个服务器
- 许多服务的 UI 可以被部署或者载入到同一个页面（服务混搭）

## 服务的组成

如果一个服务是一个逻辑构造/边界并且可以部署成多个部分，那么服务到底是怎么组成的呢？
我觉得自治服务是由内部自治的组件或者微服务，共同协作来支持服务（跟业务功能匹配）的各种功能和用例。
微服务是逻辑服务的实际实现细节。
我们关注和谈论的焦点是服务和服务支持的业务功能和用例，我们不要过度关心实现细节，比如它们对应的微服务（或者自治组件），这是件好事，因为微服务比起服务本身来说很不稳定（比如假设需要修改一个读模型的微服务，因为它太慢或者要返回更多的数据）。焦点放在服务而不是实现细节上让我们更容易的对微服务进行重写（只要协议是稳定的）或者扩展（需要运行另一个版本的微服务的情况）。

每个微服务都可以有一到多个应用或者网关的终端（endpoint），让使用者可以跟微服务沟通，终端可以是 Http 形式比如返回 HTML 格式的 UI，可以是 REST 形式执行查询或者处理一个命令，可以是消息队列形式从队列中捕获消息（比如命令）并且异步的处理。
终端可以是使用常规的 Java 或者 .NET 等来实现，这对于本地 IT 操作（比如集成网关）可以不使用远程方法调用（PRC）。
微服务组成了服务，服务拥有特定的领域模型并负责该领域对应的功能，可以是写模型聚合，可以是视图模型投影，可以是报表，也可以是 CRUD，最终以微服务的形式呈现，并且发布消息到消息总线、订阅主题（Topic）或者暴露 Atom+Pub 格式的 REST 终端，这样其他的微服务才可以订阅或者消费它们。

## 微服务应该多小？

在第三部分，基于 Pat Hellands [Life Beyond Distributed Transactions ? – An Apostate ‘s Opinion](http://www-db.cs.wisc.edu/cidr/cidr2007/papers/cidr07p15.pdf) 一文的描述，我们形成了一个经验规则：1 use-case = 1 transaction = 1 aggregate。这意味着变更操作（也就是一个有副作用的操作，比如修改业务数据）通常只应该影响一个聚合，以保证在不需要分布式事务的情况下可扩展并保证数据一致性。

换种方式说：微服务是服务沿事务和一致性边界划分的。

在读取端(read side)，比如报表和查询，我们需要创建最小的微服务负责维护读模型（比如通过处理 CQRS 写端微服务发布的事件），执行查询或生成报表，熟悉 CQRS 的读者知道读模型/查询/报表总是需要牵扯多个聚合实例，有时候还会返回超过一个聚合类型的内容，只需要确保这些被封装在一个可部署的单元（比如微服务）内部。

这并不意味着微服务只需要关注上面的提到的这一点，基于性能、可扩展性、一致性、可用性的需求，我们会选择在一个微服务里绑定更多的概念进而把微服务拆成更小的部分（可以理解为把服务多维度的拆解成更小的微服务）。

![逻辑 SOA 组件](https://github.com/hotjk/translation/blob/master/microservices/Image/Logical-SOA-components.png?raw=true)

并且没有强制要求服务的内部组件（微服务）只能使用事件来进行通信，绝对可能也有理由考虑那些可以应对分布式数据的存储的平台，比如 [NuoDB](http://www.nuodb.com/)

我们也没有说微服务必须或者应该被部署为自己的独立进程，我觉得微服务是逻辑部署单元，这意味着他们可以被独立部署，但只在需要的时候。

独立进程单元部署带来的开销包括序列化、反序列化、安全、通信、维护、配置、部署、监控等等，我们只需要在你有典型的非功能性需求强制要求部署到独立的进程单元时才接受这些代价。

## Conway 法则

最后，我们还没有提到 Conway 法则，它描述了机构怎样借助自治服务和微服务来工作：

> “机构设计出来的系统受制于机构的组织和沟通方式。”

话句话说：团队的工作方式直接反应出团队的架构方式。
如果把项目分成三个组，你会得到三个服务，如果把编译程序分成五部分，你会得到一个五步来完成的编译程序。

大多数想通过自治服务和微服务取得成功的机构，典型情况下它的团队是按应用程序划分的而不是与服务匹配，更坏的情况是，很多机构的团队是按产品划分的，每次开新项目都会设立一个新团队。

Jay Kreps 描述了这个问题：

![软件主要是人力资本，失去团队比丢失代码更可怕](https://github.com/hotjk/translation/blob/master/microservices/Image/software-is-mostly-human-capital.png?raw=true)

这篇文章到此为止，后面我们希望谈谈怎样集成遗留系统或第三方应用。
