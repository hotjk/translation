本文翻译自 Kamil Grzybek 的博客文章，原文地址：
> https://www.kamilgrzybek.com/design/modular-monolith-integration-styles/

![Modular Monolith Integration Styles](https://github.com/hotjk/translation/blob/master/microservices/mm/modular_monolith_integration_styles_logo-825x510.jpg)
 
# 模块化单体：集成方式

这篇文章是关于模块化单体架构的系列文章的一部分：

1. [模块化单体：基础](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-primer.md)
2. [模块化单体：架构驱动因素](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-architectural-drivers.md)
3. [模块化单体：架构实施](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-architecture-enforcement.md)
4. 模块化单体：集成方式
5. [模块化单体：以领域为中心的设计](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-domain-centric-design.md)

## 介绍

大型系统中，没有任何模块或应用程序可以 100% 隔离地工作。为了提供业务价值，各个部分必须以某种方式相互集成。让我在这里引用[《Thinking in Systems: A Primer》](https://www.amazon.com/Thinking-Systems-Donella-H-Meadows/dp/1603580557)一书中 Donella H. Meadows 对系统这个概念进行的概括性定义：

> 系统是一组相互关联的元素，它们以某种方式连贯地组织起来，以实现某些目标。如果仔细查看该定义一分钟，您会发现系统必须由三种事物组成：元素、互连以及功能（或目的）。

系统集成的概念定义如下（[wiki](https://en.wikipedia.org/wiki/System_integration)）：

> …将不同的计算系统和软件应用程序在物理上或功能上连接在一起并作为一个协调的整体的过程。

从上面的定义可以看出，为了提供一个满足其功能需求的系统，我们必须整合元素以形成一个整体。在本系列的前几篇文章中，我们讨论了这些元素的属性，我们把这些元素称为模块。

在这篇文章中，我只想讨论缺失的部分：模块化单体架构中模块的集成方式。

## 《企业集成模式》

这篇文章的标题并非偶然。这听起来很像 Gregor Hohpe 和 Bobby Wolf 的好书 [《企业集成模式》](https://www.amazon.com/o/asin/0321200683) 的第 2 章。这本书被认为是有关系统集成和消息传递的圣经。本文从书中汲取了一些知识，并将其与模块化单体架构相关联。

无论如何，每个对集成感兴趣的人，都推荐阅读 [https://www.enterpriseintegrationpatterns.com/](https://www.enterpriseintegrationpatterns.com/) 站点上提供的书籍或材料。

## 集成方式

### 集成标准

就像自然界中的所有事物一样，每种集成方式都有其优点和缺点。因此，我们必须定义标准，我们将根据这些标准来比较所有方式。然后，根据该标准，我们将决定选择那种集成方式。我们的标准是：耦合性、复杂性、数据时效性。

#### 1. 耦合性

耦合是衡量 2 个模块相互依赖程度的指标 ([wiki](https://en.wikipedia.org/wiki/Coupling_(computer_programming)))：

> 耦合度是软件模块之间的相互依赖程度；耦合度是衡量两个例程或模块的紧密程度；耦合度是模块之间关系的强度。

如果您已阅读本系列的前几篇文章，您就会知道模块化设计最重要的特征之一就是独立性。 因此，很容易猜到耦合性是集成方式重要的标准之一。

#### 2. 复杂性

评估集成方式的第二个标准是复杂度。一些集成方法很简单，只需要很少的工作，易于理解和使用。也有些集成方式更复杂，需要更多的承诺、知识和纪律。

#### 3. 数据时效性

最后一个标准是数据时效性，从一个模块决定共享某些数据，到另一个模块获得该数据之间的时间长短。这意味着在给定模块中的状态更改后，其他相关模块将在多长时间内会收到此更改，当然这个时间越短越好。

现在我们知道了所有最重要的标准，让我们讨论 4 种集成方式：文件传输、共享数据库数据、直接方法调用和消息传递。

### 文件传输

第一个方式是使用常规的文件来集成我们的模块。这样的文件必须从源模块导出并导入到目标模块中。 这可以通过以下三种方式：

- 手动，用户手动导入/导出
- 自动，文件由系统自动导入和导出
- 混合，文件在一侧自动导入/导出，另一侧是手动导出/导入

![集成方式-文件传输](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith-Integration-Styles-File-Transfer-768x424.jpg?raw=true)

使用文件传输来集成的关键之一是确定文件的格式。这是两个以这种方式集成的模块之间唯一的依赖。您可以将文件视为由文件系统承载的大消息。出于这个原因，可以认为这种集成方式的耦合非常低。

至于这种方式的复杂度，可以说是平均水平。一方面，在这个时代生成特定格式的文件并不困难。另一方面，上传到共享资源、管理文件、处理重复项等是复杂和耗时的。

从数据时效性的角度来看，通过文件进行模块集成很慢（更不用说手动导出/导入了）。大多数情况下，它以某种时间间隔来进行批次处理，通常是在晚上。因此，延迟可能是一天、一周甚至更长时间。

老实说，我常常见到系统之间的使用文件共享来集成，但从没在单体应用内部见过这种集成方式。为了本主题的完整性，我还是描述了这种集成方式。单体应用最流行的集成方法是共享数据库数据。

### 共享数据库数据

在《企业集成模式》这本书中，这种集成方法被叫做共享数据库，但我认为这个名字不合适。共享数据库并不总是意味着共享数据，因为模块可以将其数据存储在单独的表中（通常是放在不同的数据库scheme）。在我看来，共享数据库数据是一个更好的术语。

![集成方式-共享数据库数据](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith-Integration-Styles-Shared-Data-768x543.jpg?raw=true)

在共享数据库数据方式下，多个模块共享了数据库中的一组数据，这种方式下，数据始终是一致的，因为它们操作的就是同一份数据。如果模块 A 向表 X 写入数据，则模块 B 可以在数据库事务完成后立即读取到数据。

这种解决方案的复杂程度非常小。如今每个应用程序/模块都需要数据库，因此使用这种方法没有增加任何额外的负担。

乍一看，这种方式很完美。但是，它最大的缺点是耦合度非常高。通过共享数据，模块共享了它们的状态，从而将它们耦合在一起。高耦合意味着模块没有自主权。此外，对数据库结构甚至数据本身的一点点改变都可能在没有通知的情况下破坏另一个模块。这意味着必须斟酌和协调每次的数据库更改。数据库成了变化的瓶颈。整个解决方案也不再是[可演进的](https://www.thoughtworks.com/evolutionary-architecture)。

共享状态还有另一个明显的缺点，创建一个统一的数据模型来满足所有模块的需要几乎是不可能的。尝试使用统一的数据模型通常会导致模型模棱两可，最终难以理解、开发和维护。

要想再保持相同程度的数据时效性性的同时减少耦合，我们可以使用直接方法调用。

### 直接方法调用

第三种选择是直接调用想要集成的方法。借助封装机制，模块仅公开需要的方法，其他方法都是封闭的。通过这种方式，我们模块的状态不会像共享数据库数据方式那样全部暴露给外部。得益于这一点，调用者无法从外部破坏任何东西。

![集成方式-直接方法调用](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith-Integration-Styles-Direct-Call-1-768x325.jpg?raw=true)

不共享数据意味着每个模块都有自己的数据，数据可以位于按 schema（table）区分的同一个数据库实例中，甚至每个模块可以有一个以不同技术创建的独立数据库实例。在使用同一个数据库实例的场景中，保持数据真正隔离很重要。这意味着来自不同模块的表之间没有约束，也没有跨越两个模块对应的不同数据库表的事务。

调用者和被调用者都应该将彼此视为外部用户。两个模块都将使用不同的概念模型。因此需要使用[反腐败层 (ACL)](https://docs.microsoft.com/en-us/azure/architecture/patterns/anti-corruption-layer)。 在调用方，它可以被实现为[网关（gateway）](https://martinfowler.com/eaaCatalog/gateway.html)，在被调用方，它可以被实现为[门面（facade）](https://en.wikipedia.org/wiki/Facade_pattern)。这样，我们保留了模块的封装性。

在分布式系统中，直接方法调用就是[远程过程调用 (RPI/RPC)](https://en.wikipedia.org/wiki/Remote_procedure_call)。不幸的是，这种技术在微服务架构中经常使用，并导致所谓的[分布式单体](https://www.simplethread.com/youre-not-actually-building-microservices/)反模式。由于调用始终是同步的，这是一种时间耦合，调用方和被调用方必须同时可用。在单体应用系统中，时间耦合不是问题，因为单体内的方法本身就是同时可用的。在微服务架构下就很糟，时间耦合降低了架构独立开发独立部署的能力。阅读本系列中有关架构驱动因素的[其他文章](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-architectural-drivers.md)以了解更多详细信息。

直接方法调用集成方式似乎是一个好的选择，但也有一些缺点。首先，调用是同步的，所以调用者必须等待结果。其次，调用模块时需要明确知道要调用的模块和调用的意图，这当然是很直接的依赖关系。虽然直接方法调用的耦合低于共享数据库数据，但耦合仍然存在。如果我们想避免这些缺点，我们可以使用最后一种集成方式：消息传递。

### 消息传递

文件传输集成方式有一个很大的优势，它不会在模块之间建立依赖。但是也有一个很大的缺点，大多数情况下的数据时效性是不可接受的。消息传递没有这个缺点。数据时效性当然不如直接方法调用，因为它是异步通信，但可以确信的是在大多数情况下还是可以接受的。

![集成方式-消息传递（内存中）](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith-Integration-Styles-Messaging-in-memory-768x364.jpg?raw=true)

![集成方式-消息传递（进程间）](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith-Integration-Styles-Messaging-separate-process-768x577.jpg?raw=true)

适当地使用消息传递来实现[事件驱动架构](https://en.wikipedia.org/wiki/Event-driven_architecture)可以解除模块之间依赖关系。模块通过事件集成，不过这些事件不是[领域事件（Domain Event）](https://martinfowler.com/eaaDev/DomainEvent.html)，因为领域事件是本地的，应该封装在给定的[边界上下文](https://martinfowler.com/bliki/BoundedContext.html)中。集成事件仅包含必不可少的信息，避免出现[胖事件](https://verraes.net/2019/05/patterns-for-decoupling-distsys-fat-event/)。集成事件应该尽可能小，因为它们是给定模块提供的合约的一部分。合约越小，就越稳定 -> 其他模块更改的频率越低。

异步处理导致只能使用[最终一致性](https://en.wikipedia.org/wiki/Eventual_consistency)，不过最终一致性也有性能优势，可伸缩性（可扩展性）更好，更可靠。

消息传递的缺点是什么？首先，异步导致了整个系统的状态只能是最终一致。这就是为什么明确定义模块的边界如此重要，在模块化单体架构里明确模块边界几乎和在微服务架构里一样重要。模块化单体架构的优势在于更容易应对边界的变更。

消息传递的第二个缺点是复杂。为了提供异步处理和事件驱动架构，我们需要某种事件总线。它可以是内存中的代理也可以是单独的服务组件（例如 [RabbitMQ](https://www.rabbitmq.com/)）。此外，我们还需要用于内部处理的作业处理机制：发件箱、收件箱、内部命令消息。需要编写一些这种基础设施代码。幸运的是，这是一个通用问题，我们只需做一次，并且有很多库和框架支持。

### 比较

下面我将从耦合、数据时效性和复杂性三个角度对三种集成方式进行比较。

![比较-耦合与复杂性](https://github.com/hotjk/translation/blob/master/microservices/mm/Comparison-Low-Coupling-vs-Complexity-2-768x593.jpg?raw=true)

![比较-耦合与数据时效性](https://github.com/hotjk/translation/blob/master/microservices/mm/Comparison-Coupling-vs-Data-Timeliness-768x593.jpg?raw=true)

我们可以从这些图表中看出什么？首先，没有一种方式是完美的。我们还可以看到一些启发：

- 如果系统非常非常简单（[本质复杂度](https://wiki.c2.com/?EssentialComplexity)低），并且你不关心模块化，请选择简单性，使用共享数据库数据方式
- 如果系统很复杂（本质复杂度很高），你必须关心模块化，有以下选项：
    - 如果您更喜欢模块高度自治，并且模块之间的最终一致性是可以接受的，请选择消息传递
    - 如果您非常关心性能、可靠性、可伸缩性（可扩展性），请选择消息传递
    - 如果您必须具有强一致性，不需要模块高度自治，或者数据时效性是主要因素，请选择直接方法调用

此外，您还可以混合使用不同的集成方式。大多数情况下，当我们谈论模块化架构时，最好同时使用直接方法调用和消息传递。有些模块可以同步通信，有些模块可以异步通信，按需选择。

## 总结

正如我在开头所说，没有任何模块、组件或系统是完全孤立的。我们必须遵循“分而治之”的原则，所以我们将解决方案分成更小的部分，但最后我们必须将这些部分整合在一起以创建一个系统。

让我们再次总结所有四种集成方式：

- 文件传输 – 低耦合但数据时效性难以接受，这种方式在单体应用中不切实际
- 共享数据库数据 – 最简单、快速但将模块耦合在一起
- 直接方法调用 – 提供比共享数据库数据更低的耦合，模块封装，相对简单
- 消息传递 – 确保最低的耦合、模块化和自主性，但以复杂性为代价

选择哪种集成方式？还是跟以前一样，“视情况而定”。但是，我们至少在某种程度上解释了它们。再次重申[没有银弹](https://en.wikipedia.org/wiki/No_Silver_Bullet)。
