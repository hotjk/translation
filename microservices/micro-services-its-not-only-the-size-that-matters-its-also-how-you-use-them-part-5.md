Microservices: It’s not (only) the size that matters, it’s (also) how you use them – part 5

[第一部分](https://github.com/hotjk/translation/blob/master/microservices/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-1.md)
[第二部分](https://github.com/hotjk/translation/blob/master/microservices/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-2.md)
[第三部分](https://github.com/hotjk/translation/blob/master/microservices/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-3.md)
[第四部分](https://github.com/hotjk/translation/blob/master/microservices/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-4.md)
[第五部分](https://github.com/hotjk/translation/blob/master/microservices/micro-services-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-5.md)

本文翻译自 Jeppe Cramon 在 [tigerteam.dk](https://www.tigerteam.dk) 的博客文章，原文地址：

> https://www.tigerteam.dk/2015/microservices-its-not-only-the-size-that-matters-its-also-how-you-use-them-part-5/

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
