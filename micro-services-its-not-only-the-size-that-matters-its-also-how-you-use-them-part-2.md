

Microservices: It’s not (only) the size that matters, it’s (also) how you use them – part 2

> https://www.tigerteam.dk/2014/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-2/

在 "Micro services: It’s not (only) the size that matters, it’s (also) how you use them – part 1" 一文中，我们讨论了使用代码行数来确定服务大小是粗暴的，用来确定服务是否具备正确的职责也是完全无意义的。

我们还谈到了服务双向同步通信导致的强耦合问题和其他烦恼：
  导致通信相关的耦合（因为数据和逻辑不总在一个服务内），也导致服务合同、数据、功能耦合以及网络通信的高延迟时间
  分层耦合（持久化不总在一个服务内）
  临时耦合（当服务依赖的其他服务无法访问时服务不可用）
  事实是服务依赖其他服务减少了自治能力也让服务不可靠
  所以这些导致没有可靠消息和事务时复杂的逻辑补偿

![可复用的服务，双向同步通信和耦合](http://www.tigerteam.dk/wp-content/uploads/2014/03/reusable-services-and-coupling.png)

如果我们组合同步双向通信的小服务也好，微服务也好，依据 1 class = 1 service 的建模方式，我们实际上倒退回了 90 年代的 Corba、J2EE、分布式对象的年代。

不幸的是，新一代的开发人员并没有分布式对象的使用经验，因此也参与过此类项目去了解这些想法是多么可怕，他们在重复这段历史，只是换了一些新的技术，比如用 HTTP 代替了 RMI 和 

IIOP。

Jay Kreps 非常恰当的概括了目前微服务使用双向通信的方式：

![Jay Kreps - 微服务 == 时髦人士的分布式对象](http://www.tigerteam.dk/wp-content/uploads/2014/03/Jay-Kreps-Microservices-equals-distributed-objects.png)

仅仅因为微服务倾向于使用 HTTP、JSON、REST 并没有填补远程通信的劣势。新手常常忽略的分布式计算的劣势可以归纳为 8 个分布式计算谬论：

他们相信：
网络可靠。不管怎样，任何一个碰到过服务器连接断开，或者因为路由器、交换机、WIFI 等问题导致 Internet 无法连接的人，或多或少都知道这是个谬论。
延迟是零。网络调用相对于进程内调用的高额成本常常被忽视。带宽有限，相对于以前纳秒级调用网络调用的延迟是毫秒级的。连续执行的调用越多，整体的延迟越差。
带宽无限。实际上，即使 10G 带宽也比进程内内存调用慢的多，在固定带宽下，调用发送的数据越多，服务越小，调用的次数越多，影响越大。
网络安全。国家安全局会告诉你为什么这是个谬论。
拓扑不变。现实不是这样，产品环境的服务将经历运行环境的不断变化。旧的服务器会升级或替换，IP 地址也会变，防火墙等网络设备也会重新配置。
单一管理员。大规模安装配置需要多个管理员，网络管理员，Windows 管理员，Unix 管理员，数据库管理员等。
同类网络。多数网路有不同操作系统下支持各种网络协议的各种品牌的网络设备组成的。

以上 8 个分布式计算谬论远没有完，如果你还有兴趣，Arnon Rotem-Gal-Oz 做了更详细的回顾。

有没有服务间双向同步调用的替换方案？

Pat Hellands 在 "Life Beyond Distributed Transactions(交易) – An Apostate’s Opinion" 文中基本给出了答案。
文中 Pat 谈到理性的人不会使用分布式事务来协调事务边界。有很多理由不使用分布式事务：

事务启动时会锁定资源。服务应该是自治的，因此如果另一个服务通过一个分布式事务锁定了某个资源，就违反了服务自治规则。
不能期望服务在制定的时间内完成。服务是自治的，因此能自己决定怎样以及何时执行。这意味着服务链中最弱的一环决定了服务链的强度。
锁定防止其他交易的完成他们的工作。
锁定导致不能扩展。如果事务花费 200 毫秒锁定了一张表，那么服务最多每秒只能接受 5 个并发事务。增加再多的机器都不能改变每个事务锁定表格的时间。
2 阶段/3 阶段/ X 阶段提交的分布式事务是脆弱的设计。即使 X 阶段提交分布式事务以牺牲性能为代价解决了协调跨事务边界更新的问题。
仍然还有很多将 X 阶段提交留着未知状态的错误情景。（比如两阶段提交在提交阶段被中断，意味着一些参与者已经承诺了他们的修改，而另一些则没有。如果一个只有参与者失败了，将处于提交不可用的尴尬状态。）

![两阶段提交协议流程](http://www.tigerteam.dk/wp-content/uploads/2014/03/2-phase-commit-protocol-flow.png)

如果分布式事务不是解决方案，解决方案是什么？

解决方案分三步：

怎么拆分数据和服务
怎样发现数据和服务
数据和服务间如何通信

如果拆分和标记数据和服务

Pat Helland 认为，数据必须被集中成实体，实体需要限定大小已保证在事务范围呢的一致性。
这就意味着实体不能突破单台机器的限制（跨机器的一致性保证需要分布式事务，这是我们之前希望避免的）。这也需要实体相对于更改实体的用例来说不能太小，如果实体太小，我们就又回到了需要使用分布式事务来保证跨服务更新的老路上。

经验法则：一个事务只处理一个实体。

来个真实世界的例子：

我之前一个项目可以做反面教材，为了重用把服务分的过细，结果服务稳定性、事务性、低耦合、低延迟都失去了。

客户想确保最大化的重用两个领域概念，LegalEntity 和 Address，任何LegalEntity需要使用地址的场合都要用到 Address，比如家庭地址、工作地址、邮件、电话、手机、GPS位置等等。为了协调创建、更新、读取以及确保重用，引入一个 Task Service，"Legal Entity Task Service" 将会协调 "Legal Entity Micro Service" 和 "Address Micro Service"，"Legal Entity Task Service" 是我们创建的一个 Task Service 的角色，但是这并没有解决最重要的事务问题。

创建一个 LegalEntity，比如个人或者公司，我们首先通过 "Legal Entity Micro Service" 生成一个 LegalEntity，同时使用 "Address Micro Service" 生成了一个或多个 Address（"Legal Entity Task Service" 中的 CreateLegalEntity()决定了创建几个 Address），每个 Address 都有从 "Address Micro Service" 的 CreateAddress() 方法返回的 AddressId，"Legal Entity Micro Service" 的 CreateLogalEntity() 方法通过 "Legal Entity Micro Service" 里的 AssociateLegalEntityWithAddress() 方法将 LegalEntityId 和 AddressId 关联起来。

![不正确的微服务](http://www.tigerteam.dk/wp-content/uploads/2014/03/Bad-microservices-Create.png)

从上面的序列图，我们清楚的看到了各种级别的深度的耦合，如果 "Address Micro Service" 没有应答，就不能创建 LegalEntity。这种方案的延迟很高，因为有太多的远程调用。使用并行调用可以减少延迟，但是这种小的优化解决不了问题的本质，事务问题仍然存在。

假设 CreateAddress() 或者 AssociateLegalEntityWithAddress() 中的一个调用失败，我们就留下了一条脏记录，我们创建了一个 LegalEntity 但是其中一个 CreateAddress() 调用失败，导致系统数据不一致性，因为并不是所有的数据都创建成功。即使我们成功的创建了 LegalEntity 以及相关的全部地址，也会出现某些时候获取 LegalEntity 时无法完整获得该实体全部 Address 的情况，这也造成系统的不一致性。

这种服务的组合调用造成了 "Legal Entity Micro Service" 里的 CreateLegalEntity() 方法不堪重负，现在他得负责在调用失败时重试以及产生的一系列清理工作（补偿）。当某个服务的清理工作失败，你想怎么做？"Legal Entity Micro Service" 里的 CreateLegalEntity() 在尝试重试一个失败的服务调用或者执行清理工作时发生服务器物理故障怎么办？开发人员能保证 CreateLegalEntity() 方法的正确实现么？ 开发者能确保 CreateAddress() 方法和 AssociateLegalEntityWithAddress() 方法是幂等的？如果不能，重试服务调用时会不会创建两个 Address 对象或者把 Address 跟 LegalEntity 关联了两次？

事务性问题可以通过研究用例进而对重用粒度重新评估来解决。

LegalEntity 和 Address 服务的设计需要架构团队设计一个合乎逻辑的标准模型，并从中发掘出什么是可重用的服务。这种方式的问题是，一个标准数据模型没有考虑数据是如何使用的，也就是没有考虑该数据用例。究其原因，数据修改/创建的方式决定了事务的边界，也就是我们说的一致性界限。根据经验，在一个事务或者说一个用例内会发生修改的数据，应该有紧密相关且有统一的所有权。

所以我们的经验法则扩展为：1 usercase = 1 transaction = 1 entity

把地址提升为服务允许直接访问是我们犯的第二个错误。可以说，把重心放在重用带来了一种必须把地址包装成服务的感觉，这是在假设每个人都需要地址服务并且随时希望修改城市名称或者邮政编码，也许会有合理的原因将一些信息集中提供服务，但是每件事儿都做成这样成本就太高了。

数据模型看起来像这样：

![不好的微服务数据模型](http://www.tigerteam.dk/wp-content/uploads/2014/03/Bad-microservices-data-model.png)

如图中模型，LegalEntity 和 Address 的对应关系是共享直接关联，这就表示两个 LegalEntity 可以共享一个 Address 实例。尽管这种情况不会发生，因为两者是一种直接组合关联，更像一种父子关系。如果 LegalEntity 对象 被删除，Address 对象也就没有存储的必要了，父子关系表达了 LegalEntity 和 Address 密切的属于一个整体，他们被共同创造，一起改变，一起被使用。

这意味着我们不应该有两个实体，实际上只有一个实体，LegalEntity，一个或多个 Address 对象紧密的与实体关联。Pat Hellands 的实体一次是从领域驱动设计（DDD）中得来，DDD 还包括更丰富的词汇：

实体（Entity）-- 描述一个可以用标识而不是它的数据来辨识的对象，比如 Legal Entity（它的子类 Person 用社会安全号来标识，子类 Company 用增值税税号来标识，等等）

值对象（Value Object）-- 描述一个可以用数据而不是标识来辨识的对象，比如地址、姓名、Email地址，两个同样类型的值对象，具备同样的值就以为着两者相等。值对象不会独立存在，它总是作为一部分被关联在实体上，值对象作为实体的数据而存在。

聚合（Aggregate）-- 一群有着复杂关系的对象串在一起，聚合用于保证其关联对象的一致性。聚合也用于锁控制，以及在分布式系统中确保事务一致性。
  聚合中选择一个实体作为根并要求只能通过聚合根访问聚合内的对象。
  聚合通过 ID 来唯一标识（常常是 UUID 后者 GUID）。
  聚合之间通过 ID 来引用，聚合之间绝对不要用内存指针或者关联表来引用。（下篇文章会再次谈到）

从描述中我们能断定 Pat Helland 说到的实体在 DDD 的官方词汇里叫做聚合，DDD 的词汇更丰富，所以以后我们将继续使用 DDD 的词汇，如果你对聚合感兴趣，推荐阅读 Gojko Adzic 的文章。

通过对用例的分析（LegalEntity 和 Address 是同时创建和修改），再加上 DDD 的词汇（LegalEntity 是实体也是聚合根，Address 是值对象），现在可以重新设计数据模型（也被称为领域模型）

![LegalEntity 微服务 -- 更好的模型](http://www.tigerteam.dk/wp-content/uploads/2014/03/LegalEntity-Microservice-better-model.png)

上图的设计中，Address 中不再包含 AddressId，因为值对象不需要 ID 来标识。

LegalEntity 仍然有 LegalEntityId，并且在使用 LegalEntity 微服务时必须通过 LegalEntityId。

在新的设计中，Address 服务已经废弃，只留下 "Legal Entity Micro Service"。

![更好的 LegalEntity 微服务](http://www.tigerteam.dk/wp-content/uploads/2014/03/LegalEntity-microservice.png)

新的设计完全解决了事务问题，因为例子中只有一个服务。还有很多可以改进的，我们还没有谈到服务间如何通信，当处理过程或用例要跨越多个聚合或服务时，确保整个服务的协调性和一致性，

这篇文已经太长了，剩下的等下文再说。
