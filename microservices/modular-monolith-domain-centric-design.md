本文翻译自 Kamil Grzybek 的博客文章，原文地址：
> http://www.kamilgrzybek.com/design/modular-monolith-domain-centric-design/

![Modular Monolith Domain-Centric Design](https://github.com/hotjk/translation/blob/master/microservices/mm/ModularMonolith_Design_logo-825x510.png?raw=true)
 
# 模块化单体：领域中心化设计

这篇文章是关于模块化单体架构的系列文章的一部分：

1. [模块化单体：基础](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-primer.md)
2. [模块化单体：架构驱动因素](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-architectural-drivers.md)
4. [模块化单体：架构实施](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-architecture-enforcement.md)
5. [模块化单体：集成方式](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-integration-styles.md)
6. 模块化单体：领域中心化设计

## 介绍

在本系列的前几篇文章中，我介绍了什么是模块化单体，它的架构是什么样的，以及如何实施该架构。然后我描述了这个架构的架构驱动因素和模块之间的集成方式。

在这篇文章中，我想更深入一层描述如何设计这样的架构。我们这次还不会实施这种架构，我们将只专注于与技术无关的设计阶段。

## 领域中心化设计

正如模块化单体架构的名称所暗示，架构的设计必须高度模块化。因此，系统必须具有提供整个业务功能的自治模块。这就是为什么在这种情况下以领域为中心的架构和设计是自然的选择。

此外，如我们所知，模块必须有明确定义的接口。模块之间的所有通信只能通过这些接口进行，这意味着每个模块都必须高度封装。

让我们从高层视角看看这样的架构是怎样的：

![模块化单体-领域中心化设计](https://github.com/hotjk/translation/blob/master/microservices/mm/ModularMonolithDesign-768x603.jpg?raw=true)

从高层视角来看，领域中心化设计与领域中心化架构有相似之处。无论是从整个系统（[系统架构](https://en.wikipedia.org/wiki/Systems_architecture)）还是从单个模块（[应用程序架构](https://en.wikipedia.org/wiki/Applications_architecture)）看，都很相似。

可以看到与[六边形架构](https://alistair.cockburn.us/hexagonal-architecture)的相似之处：

- API：主要适配器
- 模块 API：主要端口
- 辅助端口及其适配器（与数据库、事件总线、其他模块通信）

![模块化单体-六边形架构视角](https://github.com/hotjk/translation/blob/master/microservices/mm/MMDesign_Hexagonal-1-768x544.jpg?raw=true)

如果我们仔细观察，这与 [洋葱架构](https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/)和[整洁架构](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)没有什么不同，都是将领域放在中心，并且遵守这条依赖规则：

> 源代码依赖性必须仅指向内部，指向更高级别的策略。

![模块化单体-整洁/洋葱架构视角](https://github.com/hotjk/translation/blob/master/microservices/mm/MMDesign_Clean_Onion-768x681.jpg?raw=true)

让我们尝试一个个介绍上面提到的概念。

### API

API 是我们系统的入口点。通常实现为接受 HTTP 请求并返回 HTTP 响应的 Web 服务（SOAP/REST/GraphQL）。

API 的主要职责，不，API 的唯一职责是将请求转发到适当的模块。它相当于微服务架构中的 [API 网关](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/direct-client-to-microservice-communication-versus-the-api-gateway-pattern)，只是把网络调用替换为内存中的模块间调用。

API 应该非常精简。不应该包含逻辑，无论是应用层逻辑还是业务层逻辑都不应该包含。API 的一切都应该只与处理 HTTP 请求和路由相关。

### 模块

每个模块都应该被视为一个单独的应用程序。换句话说，它是我们系统的一个子系统。因此它将拥有自主权。它将与其他模块（子系统）保持松散不耦合。这意味着每个模块都可以由单独的团队开发，这常常也是促使我们使用微服务架构的[架构驱动因素](https://www.informit.com/articles/article.aspx?p=2738304&seqNum=4)。

此外，我们将能够轻松地将特定模块提取到单独的运行时组件中（拆分单体）。当然只有在必要时才这么做，因为这并不是我们架构的目标，这只是模块化的一个很好的副作用。

由于模块应该是面向领域的（参见 DDD 战略模式集中的[边界上下文](https://martinfowler.com/bliki/BoundedContext.html)概念），在模块层面我们可以再次使用以领域为中心的架构。

模块的架构如下：

![模块架构](https://github.com/hotjk/translation/blob/master/microservices/mm/Module_Architecture-1.jpg?raw=true)

#### 模块 Startup API

模块的 Startup API 是一个端口/接口，通过它可以初始化给定的模块。由于给定的模块必须是自治的，它应该能够自我初始化，仅获取其操作所需的适当配置参数。这意味着我们不会在 API（或其他模块Host）中配置给定模块。我们只在 Startup 初始化它。

#### 组合根

支持模块自治还意味着给定的模块必须能够自己创建对象依赖图，应该有自己的[组合根](https://blog.ploeh.dk/2011/07/28/CompositionRoot/)。这通常意味着它将拥有自己的 [IoC](https://en.wikipedia.org/wiki/Inversion_of_control) 容器，这是非常重要的事情。不幸的是，我们常常为整系统定义一个 IoC 容器。只是用一个 Ioc 容器的方案适合小型系统，在更复杂的模块化系统里不建议这样做。

#### 模块 API

模块 API 是用于与给定模块通信的接口（主要端口）（Startup API 除外），可以通过两种方式创建模块 API：

- 传统方式：一组方法列表（CustomerService.GetCustomer、OrderService.AddOrder）
- [CQRS](https://docs.microsoft.com/en-us/azure/architecture/patterns/cqrs) 方式：发送一系列的查询和命令（GetCustomerQuery、AddOrderCommand）

我绝对是第二种 CQRS 风格的方法的粉丝，但在我看来，第一种方法也是可以接受的。

根据[模块化关键属性](https://en.wikipedia.org/wiki/Modular_programming)所说，模块 API 应该尽可能小，只公开需要的东西（不多也不少），这将使它更[稳定](https://wiki.c2.com/?StableDependenciesPrinciple)。

#### 基础设施–辅助适配器

辅助适配器（使用端口适配器架构的命名法）应该实现在这儿。辅助适配器负责与外部依赖项（进程内的和进程外的）通信，比如数据库、事件总线或者其他模块。

#### 应用 Application

与模块相关的用例在 Application 实现，从洋葱架构的视角或模块化单体架构视角，Application 都是应用程序核心边界。因为使用了以领域为中心的架构，它很好的与框架和基础设施分离。

这个领域的模型（[Domain Model](https://martinfowler.com/eaaCatalog/domainModel.html)）只适用于当前边界上下文（boundary）。这也是唯一跟我们的领域也就是所谓的企业业务规则相关的概念。

领域模型应该完全[不关心持久化](https://www.kamilgrzybek.com/design/domain-model-encapsulation-and-pi-with-entity-framework-2-2/)，使用[通用语言](https://martinfowler.com/bliki/UbiquitousLanguage.html)描述并且完全可测试。这是最重要的部分，其他部分都是为了简化这部分。

#### 分几层？

网上关于应用层的讨论很多，有人推荐清晰的层次划分（使用独立的库/包或者其他计算手段），也有人推荐将所有内容放在一起而不进行逻辑分解。

首先，请注意每个模块都会有所不同。有的领域非常复杂，有的可能只实现一些 CRUD 操作，因此应用程序架构会有所不同。

此外，模块内部或多或少也有些复杂的功能，这些功能也应该有良好的功能分离。

总之，应用层不应该使用全局处理方式，应该针对每个用例单独处理。这种方法接近[垂直切片架构](https://jimmybogard.com/vertical-slice-architecture/)，只是这次应用于模块级别，而不是整个应用程序级别。

有人说以领域为中心的架构和垂直切片是对立的。我觉得恰恰相反，两者完美互补。

![模块化单体架构风格](https://github.com/hotjk/translation/blob/master/microservices/mm/ModulesArchitecture-768x228.jpg?raw=true)

### 模块数据

每个模块必须有自己的状态，这意味着它的数据必须是私有的。我们不想使用共享数据库模式，这是实现模块自主性的关键。如果我们想知道模块的状态或改变状态，必须通过接口来完成，没有捷径。

但是，有时我们希望共享一些数据来生产报表。在这种情况下，我们可以使用单独的报表数据库，并以特殊的集成方案在模块数据库上以单独视图的形式为报表数据库提供数据，这种方案仅限于此类需求，我们最好为此创建一套直接操作数据库的 API。

### 模块集成

我在上一篇文章中详细介绍了模块集成。如您所见，模块化单体架构有两种通信方式：

- 通过事件来异步通信（[事件驱动架构](https://en.wikipedia.org/wiki/Event-driven_architecture)）。 每个模块通过事件总线发送或订阅某些事件。根据不同的需求，事件总线可以是进程内存内，也可以是进程外的组件。

- 进程内存里的同步调用。跟模块间的 API 通信类似，可以通过传统方法也可以通过 CQRS 方式（命令/查询）来实现。需要注意，集成应该显式的在消费端创建一个 [Gateway](https://martinfowler.com/eaaCatalog/gateway.html)（适配器）并在供给端创建一个 [Facade](https://en.wikipedia.org/wiki/Facade_pattern)（端口及其实现）。

### 测试

如果你考虑模块化你的系统，那你的系统一定不是个简单系统，自动化测试也就必不可少。对测试覆盖度的要求取决于系统的复杂程度、集成数量和其他因素。测试是一个广泛的主题，超出了本文的范围。这里我只想强调以下测试方式。

#### 端到端测试

端到端测试关注整个系统，从 API 到基础设施再回到 API。他们通常可以检查我们系统的各个部分，因此拥有最大的测试代码覆盖率。但是，这些测试通常还与 GUI 搅在一起。因此它们是最慢、最脆弱且最难以维护的。

#### 集成测试

集成测试这个词使用广泛，不同场合解释不同，在这里，集成测试指对给定模块（或模块之间的交互）进行综合测试。集成测试是模块的第二个消费者（只是另一个适配器）。集成测试不涉及 API 层，API 层非常薄，它们几乎覆盖了我们的整个应用程序，并且不对低级抽象对象（JSON、HTTP）进行操作。

#### 单元测试

单元测试将主要用于测试领域模型，也就是业务逻辑。由于使用以领域为中心的架构作为模块的一部分，领域模型与基础设施分离，可以轻松地在内存中进行单元测试。

## 概括

如您所见，系统模块化需要遵循正确的规则。整个系统需要不断分解成更小的片段，拼图的每一块都很重要。让我们总结一下这个架构最重要的特征：

- 在多个层面上以领域为中心
- 明确定义的集成点（接口）
- 模块封装、自治
- 可测试性 – 使应用程序和领域层独立于框架和基础设施
- 可演进性 – 易于开发和维护（添加新模块或适配器）

如果系统不需要分布式（大多数系统都不需要），也不是那种特别小的系统，领域中心化设计的模块化单体也许就挺合适。当然，这完全取决于项目的特点，因此请认真思考后再做决定。
