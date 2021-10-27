本文翻译自 Kamil Grzybek 的博客文章，原文地址：
> https://www.kamilgrzybek.com/design/modular-monolith-architectural-drivers/

![Modular Monolith Architectural Drivers](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular_Monolith_Architectural_Drivers_Promo-825x510.jpg)
 
# 模块化单体：架构驱动

这篇文章是关于模块化单体架构的系列文章的一部分：

1. [模块化单体：入门](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-primer.md)
2. 模块化单体：架构驱动
3. [模块化单体：架构执行](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-architecture-enforcement.md)
4. [模块化单体：集成风格](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-integration-styles.md)
5. [模块化单体：以领域为中心的设计](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-domain-centric-design.md)


## 介绍

在关于模块化单体架构的第一篇文章中，我重点介绍了该架构的定义和模块化的描述。回忆下模块化单体架构的定义：

- 模块化单体架构是只有一个部署单元的系统
- 模块化单体架构是以模块化方式设计的单体系统的显式名称
- 模块化意味着模块：
  - 必须独立、自治
  - 拥有提供特定功能所需的一切（按业务领域划分）
  - 被封装并具有明确定义的接口/合同
  
在这篇文章中，我想讨论一些我认为最流行的架构驱动因素，它们可以导致系统采用模块化单体架构或微服务架构。

## 架构驱动因素

一般来说，你不能简单的说某个架构比另一个架构更好。比如你不能说单体架构优于微服务架构，[整洁架构（Clean Architecture）](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)优于[分层架构（Layered Architecture）](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch01.html)，3层架构比4层架构好或者差等等。

这个规则也适用于其他技术的比较，例如 ORM 与原始 SQL、持久化数据“当前状态”与[事件溯源（Event Sourcing）](https://martinfowler.com/eaaDev/EventSourcing.html)、[贫血模型（Anemic Domain Model）]与充血域模型(https://www.martinfowler.com/bliki/AnemicDomainModel.html)、面向对象设计与函数式编程等等。

既然这样，我们如何选择架构/方法/范式/工具/库？

## 上下文为王

我们的每一个决定都是在给定的上下文下做出的。每个项目都不一样，因此每个上下文也都是不同的。这意味着某些决定在一个上下文取得巨大的结果，而在另一种上下文可能会导致毁灭性的失败。出于这个原因，在没有批判性思维的情况下使用其他人或其他公司的方法很可能会痛苦万分，浪费金钱，甚至项目失败。

![不同的项目有不同的上下文](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_Contexts-1-768x241.jpg?raw=true)

然而，上下文这个概念过于笼统，我们需要一些更具体的东西来付诸实践。这就是定义架构驱动因素这个概念的原因。 Michael Keeling 在他的[博客文章](https://www.neverletdown.net/2014/10/architectural-drivers.html)中这样定义架构驱动因素：

> 架构驱动因素是对架构有重大影响的需求集合的标准定义。

Simon Brown 在 [Software Architecture for Developers](https://softwarearchitecturefordevelopers.com/) 一书中对架构驱动因素的描述类似：

> 无论您遵循哪种流程控制方式（传统的计划驱动的流程控制方式或者轻量级自适应的流程控制方式），都有一组共同的东西真正驱动、影响和塑造最终的软件架构。

架构驱动因素是他们的总称，主要可以分为：

- 功能需求——系统解决什么问题以及如何解决问题
- 质量属性——一组决定架构质量的属性，如可维护性或可扩展性。
- 技术约束——技术标准、工具限制、团队经验
- 业务限制——预算、期限

![架构驱动因素](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Architectural-Drivers-768x364.jpg?raw=true)

最重要的是，所有架构驱动因素都相互关联的，专注于提高一个通常导致另一个降低（到处都有权衡取舍）。 

举个不太恰当的例子，注意括号里的机构驱动因素：我们有一个服务，可以在 3 秒内完成一组重要的计算（功能需求）（质量属性 - 性能）。出现了新的功能需求，计算变得更复杂了，现在计算服务需要 5 秒才能给出结果（性能下降）。想要优化到 3 秒，可以使用另一种技术来重新实现计算功能，但是我们没有时间（业务约束 - 硬期限），并且公司中还没有人会这种技术（技术约束 - 团队经验）。综合考虑后，为了提高计算性能，只能是将计算移至存储过程中进行，这会降低代码的可维护性和可读性（质量属性）。

![架构驱动因素例子](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Example-768x333.jpg?raw=true)

如您所见，软件架构是在一个个架构驱动因素之前的权衡取舍。没有哪一个是“正确”的解决方案，没有银弹。考虑到这一点，让我们看看在考虑模块化单体和微服务架构时讨论的一些流行的架构驱动因素和属性。

## 复杂度

首先我们需要考虑复杂度，这是模块化单体架构对比分布式架构的最大优势之一。[维基](https://en.wikipedia.org/wiki/Complexity)上对复杂度的定义如下：

> 复杂度表现为系统或者模块内组件通过多种方式和规则进行交互的行为，这意味着无法对各种可能的交互进行高层抽象定义，复杂度通常用于描述具有多个组成部分的事物，这些部分以多种方式进行交互，这些组成部分最终呈现出远超部分之和的[涌现（Emergence）](https://en.wikipedia.org/wiki/Emergence)
>> 涌现指复杂系统中在自我组织的过程中，所产生的新奇且清晰的结构、图案和特征。

正如您在上面看到的，复杂性与组件及其交互有关。在模块化单体架构中，模块之间的交互很简单，因为每个模块都位于同一进程中。这意味如果模块想要与另一个模块交互就需要：

- 明确知道它将请求的地址，并且该地址不会改变
- 请求只是一个简单的方法调用，不需要经过网络
- 目标模块始终可用
- 安全问题不是问题

![模块化单体的复杂度](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Complexity-768x304.jpg?raw=true)

而与之相对的分布式系统，模块/服务位于其他服务器上，模块/服务之间需要通过网络进行通信。当一个服务想要与另一个服务进行通信时，它必须处理以下问题：

- 它需要以某种方式获取目标模块的地址，因为它可能会被更改
- 通过网络进行通信，这需要使用 HTTP 和序列化等特殊协议。
- 网络可能不可用（[CAP 定理](https://en.wikipedia.org/wiki/CAP_theorem)）
- 必须确保模块之间的安全通信

当然，你可以找到这些问题的解决方案。例如，为了解决寻址问题，您可以添加 [服务注册（Service Registry）](https://microservices.io/patterns/service-registry.html) 并实现 [服务发现模式（Service Discovery pattern）](https://microservices.io/patterns/server-side-discovery.html)。 但是，这意味着向系统添加更多组件和算法，因此复杂性会迅速增加。

要了解微服务架构产生的问题，我建议您熟悉用于解决这些问题的 [模式](https://microservices.io/patterns/index.html)。这个模式列表很大，其中大部分在单体架构中根本不需要。

![分布式系统的复杂度](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Complexity-Distribiuted-System-768x304.jpg?raw=true)

综上所述，模块化单体架构绝对没有分布式系统那么复杂。高复杂性降低了可维护性、可读性和可观测性。它需要经验丰富的团队、先进的基础设施、特定的组织文化等。如果简单性是关键的架构驱动因素，那么请优先考虑单体。

## 生产力

团队交付需求的生产力可以从两个维度来衡量：在整个系统维度和单个模块的维度。

从整个系统的维度看，这件事很清楚。模块化架构不那么复杂 => 越不复杂越容易理解 => 生产力越高。从整个系统易于运行的角度来看，模块化架构可确保最高水平的生产力：只需下载代码并在本地机器上运行即可。在分布式架构中，尽管有促进这一过程的技术和工具（如 Docker 和 Kubernetes），但事情并不那么简单。

![系统运行：单体与分布式](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Productivity-768x364.jpg?raw=true)

从单个模块的维度看，我们的生产力与单个模块的开发有关。在这种情况下，微服务架构会更好，因为我们不必运行整个系统来测试一个特定的模块。

综合考虑团队的生产力，对于大多数系统来说是模块化单体的生产力高，但对于真正的大型项目（数十或数百个模块）来说微服务的生产力高。如果您的架构驱动因素是开发速度并且系统不是很大，那么更好的选择是模块化单体，从模块化单体开始，当系统不断扩展时再过渡到微服务可能是正确的举措。


