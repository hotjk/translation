本文翻译自 Kamil Grzybek 的博客文章，原文地址：
> https://www.kamilgrzybek.com/design/modular-monolith-architectural-drivers/

![Modular Monolith Architectural Drivers](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular_Monolith_Architectural_Drivers_Promo-825x510.jpg?raw=true)
 
# 模块化单体：架构驱动因素

这篇文章是关于模块化单体架构的系列文章的一部分：

1. [模块化单体：基础](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-primer.md)
2. 模块化单体：架构驱动因素
3. [模块化单体：架构实施](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-architecture-enforcement.md)
4. [模块化单体：集成方式](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-integration-styles.md)
5. [模块化单体：领域中心化设计](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-domain-centric-design.md)

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

## 场景为王

我们的每一个决定都是在给定的场景下做出的。每个项目的场景都不一样，这意味着一个决定在某个场景很成功，而在另一个场景可能会导致毁灭性的失败。出于这个原因，在没有批判性思维的情况下使用其他人或其他公司的方法很可能会费时费力还搞砸项目。

![项目有不同的场景](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_Contexts-1-768x241.jpg?raw=true)

场景这个概念过于笼统，我们需要一些更具体的东西来付诸实践。这就是定义架构驱动因素这个概念的原因。 Michael Keeling 在他的[博客文章](https://www.neverletdown.net/2014/10/architectural-drivers.html)中这样定义架构驱动因素：

> 架构驱动因素是对架构有重大影响的需求集合的标准定义。

Simon Brown 在 [Software Architecture for Developers](https://softwarearchitecturefordevelopers.com/) 一书中对架构驱动因素的描述也类似：

> 无论您遵循哪种流程控制方式（传统的计划驱动的流程控制方式或者轻量级自适应的流程控制方式），都有一组共同的东西真正驱动、影响和塑造最终的软件架构。

架构驱动因素是他们的总称，主要可以分类为：

- 功能需求 —— 系统解决什么问题以及如何解决问题
- 质量属性 —— 一组决定架构质量的属性，如可维护性或可伸缩性（可扩展性）。
- 技术约束 —— 技术标准、工具限制、团队经验
- 业务限制 —— 预算、期限

![架构驱动因素](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Architectural-Drivers-768x364.jpg?raw=true)

最重要的是，所有架构驱动因素都相互关联的，专注于提高一个因素通常影响另一个因素（生活处处都是权衡取舍）。 

举个不太恰当的例子，注意括号里的架构驱动因素。假设我们有一个服务，可以在 3 秒内完成一组重要的计算（功能需求）（质量属性 - 性能）。为了解决新的功能需求，计算逻辑变得更复杂了，现在计算服务需要 5 秒才能给出结果（性能下降）。我们想要优化到 3 秒，可以使用另一种技术或者算法来重新实现计算功能，但是我们没有时间（业务约束 - 硬期限），并且公司目前也没有会这种技术的人（技术约束 - 团队经验）。综合考虑后，我们将计算工作移至存储过程中进行来临时提升计算性能，存储过程降低了代码的可维护性和可读性（质量属性）。

![架构驱动因素的例子](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Example-768x333.jpg?raw=true)

如您所见，软件架构需要在多个架构驱动因素之间权衡取舍。没有哪一个是“正确”的解决方案，没有银弹。考虑到这一点，让我们从模块化单体架构和微服务架构视角看看一些主要的架构驱动因素。

## 复杂度

首先需要考虑的是复杂度，这是模块化单体架构对比分布式架构的最大优势之一。[维基](https://en.wikipedia.org/wiki/Complexity)上对复杂度的定义如下：

> 复杂度表现为系统或者模块内组件通过多种方式和规则进行交互的行为，这意味着无法对各种可能的交互进行高层抽象定义，复杂度通常用于描述具有多个组成部分的事物，这些部分以多种方式进行交互，这些组成部分最终呈现出远超部分之和的[涌现（Emergence）](https://en.wikipedia.org/wiki/Emergence)
>> 涌现指复杂系统中在自我组织的过程中，所产生的新奇且清晰的结构、图案和特征。

复杂性与组件及其交互有关。在模块化单体架构中，模块之间的交互很简单，因为每个模块都位于同一进程中。这意味着当一个模块与另一个模块交互时：

- 明确知道它将请求的地址，并且该地址不会改变
- 请求只是一个简单的方法调用，不需要经过网络
- 目标模块始终可用
- 安全问题不是问题

![模块化单体的复杂度](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Complexity-768x304.jpg?raw=true)

而与之相对的分布式系统，模块/服务位于不同的服务器上，模块/服务之间需要通过网络进行通信。当一个服务想要与另一个服务进行通信时，它必须处理以下问题：

- 它需要以某种方式获取目标模块的地址，因为它可能会被更改
- 通过网络进行通信，这需要使用 HTTP 和序列化等特殊协议。
- 网络可能不可用（[CAP 定理](https://en.wikipedia.org/wiki/CAP_theorem)）
- 必须确保模块之间的安全通信

当然，你可以找到这些问题的解决方案。例如，为了解决寻址问题，您可以添加 [服务注册（Service Registry）](https://microservices.io/patterns/service-registry.html) 并实现 [服务发现模式（Service Discovery pattern）](https://microservices.io/patterns/server-side-discovery.html)。 但是，这意味着向系统添加更多组件和算法，系统复杂性会迅速增加。

要解决微服务架构产生的这些问题，你需要熟练使用解决这些问题的 [模式](https://microservices.io/patterns/index.html)。模式众多，不过大多数模式在单体架构中根本不需要。

![分布式系统的复杂度](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Complexity-Distribiuted-System-768x304.jpg?raw=true)

综上所述，分布式系统显然比模块化单体系统更复杂。高复杂性降低了可维护性、可读性和可观测性。它需要经验丰富的团队、先进的基础设施、特定的组织文化等。如果简单性是关键的架构驱动因素，那么请优先考虑单体。

## 生产力

团队交付需求的生产力可以从两个维度来衡量，从整个系统层面和单个模块的层面。

从整个系统的层面看这件事很清晰。模块化架构不那么复杂 => 越不复杂越容易理解 => 生产力越高。从整个系统易于运行的角度来看，模块化架构可确保最高水平的生产力：只需下载代码并在本地机器上运行即可。在分布式架构中，尽管有促进这一过程的技术和工具（如 Docker 和 Kubernetes），但事情并不那么简单。

![系统运行：单体与分布式](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Productivity-768x364.jpg?raw=true)

从单个模块的层面看，我们的生产力与单个模块的开发有关。在这种情况下，微服务架构会更好，因为我们不必运行整个系统来测试一个特定的模块。

综合考虑下来，对于大多数系统来说，模块化单体的生产力更高，但对于真正的大型项目（有几十个上百个模块）来说，微服务的生产力高。如果您的架构驱动因素是开发速度并且系统不是很大，那么更好的选择是模块化单体，从模块化单体开始，当系统模块不断扩张时再过渡到微服务也是个不错的选择。

## 可部署性

软件系统的可部署性是指从开发阶段到生产阶段的容易程度。 在这里我们也必须考虑两种情况：整个系统和单个模块的部署。

在整个系统的层面，部署多个应用程序和部署一个应用程序那个更容易？ 当然时部署一个应用程序更容易，这是模块化单体的优势。

![模块化单体的部署](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Deployment-Modular-Monolith-768x304.jpg?raw=true)

从单个模块部署的层面看，在模块化单体中，我们总是必须部署整个系统，我们不能单独部署一个特定的模块，部署一个模块也必须协调其他模块一起部署，这是模块化单体最大的劣势之一。

![分布式系统的部署](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Deployment-Distribiuted-System-768x304.jpg?raw=true)

综上所述，如果你不介意整个系统一起部署，也不关心模块的单独部署能力，可以选择模块化单体架构。否则，请考虑分布式架构。

## 性能

性能就是系统的运行速度，通常指的是响应时间、处理持续时间或延迟等方面。

假设所有请求都按顺序处理的场景，单体架构总是比分布式系统更高效。所有模块都在同一个进程中运行，因此它们之间没有通信开销。

分布式系统具有由网络通信引起的开销，比如序列化、反序列化、加解密和发送数据包。

在真实场景中，单体系统在早期也会也会更高效。但是随着用户数、请求数、数据和计算复杂性的增加，性能可能会下降。然后我们来到微服务架构的主要驱动因素之一：可扩展性。

## 可伸缩性（可扩展性）

什么是可伸缩性（可扩展性）？ [维基百科](https://en.wikipedia.org/wiki/Scalability)的解释：

> 可伸缩性（可扩展性）是系统的属性，该属性表达了系统可以通过添加资源来处理更多工作的能力。

换句话说，可伸缩性是关于软件处理更多请求或数据的能力。

最好通过示例来说明这一点。 让我们假设我们的一个模块现在必须处理比我们最初假设的更多的请求。为此，我们必须增加负责该模块运行的资源。

增加资源的方式有两种，增加节点计算能力（称为垂直扩展）或添加新节点（称为水平扩展）。让我们从单体架构和微服务架构的角度来分析这两点。

![扩展](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Scaling-768x825.jpg?raw=true)

从上图可以看出，两种架构都支持扩展，并且两者的垂直扩展方式是类似的，差异在于水平扩展。模块化单体下，系统只能作为一个整体进行水平扩展，资源利用率低。在微服务架构中，我们只对需要扩展的模块进行扩展，从而更好地利用资源。

单个模块的实例越多，扩展导致的资源使用差异就越大。当然，如果没有大量扩展的需求，资源利用率低的单体模式带来的其他优势是否更有吸引力，我们在做架构时需要思考这个问题。 

## 故障影响

限制故障的影响范围也是架构驱动的因素。假设我们有一个非常不稳定的模块，它偶尔会导致整个过程崩溃。

在模块化单体架构的情况下，整个系统在一个进程中工作时，整个系统突然停止工作，这个系统就不可用了。在微服务架构的情况下，这个不稳定的的模块可以部署在一个单独的进程中，如果它被停止，系统的其余部分还可以正常工作。

![故障影响](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Failure-impact-768x304.jpg?raw=true)

为了提高模块化单体的可用性，您可以部署更多的服务节点，但与讨论可伸缩性时一样，与微服务架构相比，单体架构水平扩展的资源利用率较低。

## 技术异构

模块化单体无法绕过的一个关键特征是不能做技术异构。整个系统处于同一个进程中，必须运行在同一个运行时环境中。这并不意味着它必须使用相同的语言编写，因为某些平台支持多种语言（例如 .NET CLR 或 Java JVM）。 然而，多个技术栈混搭是不可能的。

![技术异构](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architectural-Drivers-Heterogeneous-Technology-768x260.jpg?raw=true)

技术异构这个特性可能是切换到微服务架构的关键因素。当然并不是所有人都有这个需求，公司通常只使用一种技术栈，受限于团队能力和软件许可证等因素，很多人从未考虑过要使用不同的技术栈实现系统的各部分。

当然，很多大公司或大项目会经常使用不同的技术栈，对特定问题选择特定的技术栈可以最大限度地提高生产力。

与技术异构相关的一个常见案例是遗留系统的维护和开发。遗留系统通常是用旧技术编写的（并且通常以非常糟糕的方式编写）。为了使用新技术，通常会创建一个新的服务/系统来实现新功能，而旧系统只将请求委托给新系统。这样，遗留系统的开发可以更快，并且更容易找到愿意使用它的人。这里的缺点是因为有两个系统，整个系统就变成分布式架构了，分布式架构带来的所有问题也随之而来。

## 总结

在这篇文章中，我们讨论讨论了一些常见的架构驱动因素，并明确说明我们系统架构风格会受许多因素的影响，一切都取决于我们所处的场景。总结下来：

- 没有一种架构是更好的或更坏的，这完全取决于你的场景和架构驱动因素
- 架构驱动因素可以分类为：功能需求、质量属性、技术约束、业务约束
- 单体架构没有分布式系统那么复杂。微服务架构需要更多的工具、库、组件、团队经验、基础设施等
- 项目初期，采用单体架构的实现会更有效率。之后如果为了满足一些特定的架构驱动因素，可以考虑迁移到微服务架构
- 单体架构系统部署更容易，但不支持模块级别的独立部署
- 两种架构都支持可伸缩行（可扩展性），但微服务扩展时的资源利用率更高
- 如果没有扩展需求，单体应用比微服务应用性能更好
- 单体系统的故障影响更大，因为一切都在同一个过程中进行。可以通过扩展来降低故障风险，但单体架构水平扩展的成本比微服务架构更高
- 单体架构注定不支持技术异构
