本文翻译自 Kamil Grzybek 的博客文章，原文地址：
> https://www.kamilgrzybek.com/design/modular-monolith-architecture-enforcement/

![Monolith Architecture Enforcement](https://github.com/hotjk/translation/blob/master/microservices/mm/Monolith_arch_enforcement-825x510.jpg?raw=true)
 
# 模块化单体：架构实施

这篇文章是关于模块化单体架构的系列文章的一部分：

1. [模块化单体：基础](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-primer.md)
2. [模块化单体：架构驱动因素](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-architectural-drivers.md)
3. 模块化单体：架构实施
4. [模块化单体：集成方式](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-integration-styles.md)
5. [模块化单体：领域中心化设计](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-domain-centric-design.md)

## 介绍

在之前的文章中，我们讨论了模块化单体架构架构以及影响架构选择的驱动因素。在这篇文章中，我想重点介绍架构实施的方法。

下面介绍的方法不仅适用于模块化单体架构，也适用于其他架构方法。然而，由于单体架构的一些特质，比如代码库的大小和变更的容易程度，架构实施对单体架构来说更为重要。

## 模型代码的偏差

假设基于当前项目的架构驱动因素，您选择了模块化单体架构，并且模块边界也被定义好了，其他的技术、方法、模块间通信方式、持久化方式等等也都选定了。一切可能都以解决方案架构文档 ([SAD](https://en.wikipedia.org/wiki/Software_architecture_description)) 的形式记录了下来，也可能你只制作了一些图表（使用 UML、C4 模型或简单的箭头和框）。总之您已经完成了足够多的前期设计工作，您可以开始项目实施阶段的第一次迭代。

一开始很简单。它没有太多功能，代码很少，易于维护，代码与模型保持一致。项目时间充裕，即使出现问题，也很容易重构。到现在为止还挺好的。

过了一阵子，问题出现了。随着功能和代码的不断增加增加，需求变更层出不穷，截止日期日益临近。我们开始抄近路了，我们的实现开始与设计明显不同。在模块化单体架构内的模块很快失去了模块化和独立性，各个模块的代码开始随意的互相调用，代码变成了一个[大泥球](https://en.wikipedia.org/wiki/Big_ball_of_mud)。

![](https://github.com/hotjk/translation/blob/master/microservices/mm/BBoM.jpg?raw=true)

George Fairbanks 在他的书[《恰如其分的软件架构:风险驱动的设计方法》](https://www.amazon.com/Just-Enough-Software-Architecture-Risk-Driven/dp/0984618104) 中对上述现象的定义如下：

> 您的架构模型和源代码不会完全一致。它们之间的差异就是模型代码偏差。

无论是先写代码再做模型，还是先设计模型再写代码，你都必须在解决方案中管理两这种表示形式。最初，代码和模型可能完美对应，但随着时间的推移，它们往往会出现分歧。代码随着功能的添加和错误的修复而不断发展。模型也随着业务和计划不断调整。当两者的演变产生不一致时，就会发生分歧。

命中注定？怎样避免？我们需要严格遵守纪律，但纪律不是一切。我们需要适当的实践和方法来控制我们的架构。那么这些方法是什么呢？

## 架构实施

检查实现是否与设计一致时，我们会使用到工具，可以通过两个方面来考察工具。

第一个方面是该工具为我们提供的可能性。正如我们所知，架构是一组不同抽象级别的规则，常常很难定义，检查它们就更难。

第二个方面是我们获得反馈的速度。越快越好，这样我们就能够更快地解决问题。我们修复某些东西的速度越快，这个错误对我们以后的架构造成的影响就越小。

下面我们将在 3 个层级进行架构实施：编译时、自动化测试和代码审查。

![架构实施方法](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architecture-Enforcement-Enforcement-tools-768x481.jpg?raw=true)

## 编译时

编译器是我们最好的朋友。它能够快速为您检查许多需要您花费很长时间才能发现的问题。人会犯错，编译器不会。既然编译器这么厉害，为什么我们很少使用编译器来确保我们的实施遵循架构规范呢？为什么不最大程度地利用编译器呢？

造成这种情形的一个源头是“public原则”。根据模块化的定义，模块应该通过定义良好的接口进行通信，这意味着它们应该被封装。如果一切都是公开的，则没有封装可言。

遗憾的是，编程社区的教程、实例项目，甚至 IDE（默认创建public类）都在鼓励 public，默认情况下，我们绝对应该将方法设为 private。如果某些东西不能设为 private，就限制它只能在模块范围内使用，禁止模块外访问。

不幸的是，在 .NET 世界我们的选择有限，唯一能做的就是将模块分离成一个单独的程序集并使用“internal”访问修饰符。也有反对者不愿意拆分程序集，坚持将所有代码放在一个项目（程序集）中，双方争论不休。

反对者坚持程序集是一个执行单元。是的，但由于我们没有其他方法来封装我们的模块，因此将项目拆分是一个明智的选择。别担心，引用检查机制可以尽可能确保你不会添加错误的依赖项（比如领域依赖基础设施）。

缺乏封装是我看到的最常见的错误之一，此类问题还有很多，比如很多人不遵循不变性原则（属性里有不必要的 set 方法），也有人不为业务对象定义强类型（原始类型痴迷）。

一般来说，我们在使用我们的语言时，应该让编译器可以捕获尽可能多的错误。这是确保系统架构实施准确的最有效方法。

![架构实施-编译时](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architecture-Enforcement-Compile-time-768x416.jpg?raw=true)

## 自动化测试

并非所有问题都能使用编译器检查出来，不过，这也不意味着我们只能做手动检查，计算机还提供了两种机制：静态代码分析和自动化测试。

### 代码静态分析

代码静态分析器是我们熟悉的一种常规方法。大家都听说过诸如 [SonarQube](https://www.sonarqube.org/sonarqube-8-2/) 或 [NDepend](https://www.ndepend.com/) 之类的代码静态分析工具。 这些工具可以自动对我们的代码进行静态分析，并提供对我们非常有用的指标信息。 当然，我们也可以把静态代码分析器连接到 CI 流程中以便定期获得反馈。

![架构实施-代码静态分析](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architecture-Enforcement-Copy-of-Compile-time-768x219.jpg?raw=true)

### 架构测试
架构测试是另一种鲜为人知但越来越受欢迎的方式。下面这些单元测试不是测试业务功能，而是在架构层面测试我们的代码库。大多数情况下，此类测试是基于专用于此类测试的库编写的。

    [Test]
    public void ValueObject_Should_Be_Immutable()
    {
        var types = Types.InAssembly(DomainAssembly)
            .That()
            .Inherit(typeof(ValueObject))
            .GetTypes();

        AssertAreImmutable(types);
    }

    [Test]
    public void DomainLayer_DoesNotHaveDependency_ToInfrastructureLayer()
    {
        var result = Types.InAssembly(DomainAssembly)
            .Should()
            .NotHaveDependencyOn(ApplicationAssembly.GetName().Name)
            .GetResult();

        AssertArchTestResult(result);
    }

我们可以通过这些测试检查很多东西。架构测试库（例如 [NetArchTests](https://github.com/BenMorris/NetArchTest) 或 [ArchUnit](https://www.archunit.org/)）功能强大使用简单。这里给出一个此类测试的[完整示例](https://github.com/kgrzybek/modular-monolith-with-ddd/tree/master/src/Modules/Meetings/Tests/ArchTests)。

![架构实施-架构测试](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular-Monolith_-Architecture-Enforcement-Architecture-tests-768x465.jpg?raw=true)

## 代码审查

如果我们无法通过计算机能力（编译器、自动化测试）来检查我们的解决方案与所选架构的一致性，我们还有最后一个工具 - 代码审查。代码审查可以检查一切计算机无法为我们做的事儿，但它有一些缺点。

第一个缺点是人们可能会犯错，因此很大概率会漏掉那些可能导致违反架构规则的点。

第二个缺点是我们需要花费大量时间进行代码审查。当然，我们不会因为要花费时间就放弃代码审查，但是在做项目的时间估算时必须考虑代码审查的时间。

结论很明显：为了确保架构稳固，我们应该尽可能多地使用计算机能力，并将代码审查作为最后一道防线。 问题是如何加强这道防线，即如何减少代码审查过程中遗漏某些东西的概率？ 我们可以使用架构决策记录（ADR）。

### 架构决策记录 (ADR)

什么是架构决策记录？让我引用一个有名的 [GitHub Repository](https://github.com/joelparkerhenderson/architecture_decision_record) 对它的定义：

> 架构决策记录 (ADR) 是一份文档，它记录了做出的重要架构决策及其上下文和结果。

这样的文档通常存储在版本控制系统中，这是 ThoughtWorks 公司著名的[技术雷达](https://www.thoughtworks.com/radar/techniques/lightweight-architecture-decision-records)推荐的方式。

我的建议是首先尽可能简单快速地描述你的决定。不用拘泥于形式，选择一个简单的模板（例如 [Michael Nygard 提出的模板](https://github.com/joelparkerhenderson/architecture_decision_record/blob/master/adr_template_by_michael_nygard.md)），在文档中列出最重要的元素：背景、决定和后果。

为什么架构决策记录能帮助到代码审查？首先，所有决定都是公开的，每个人都可以访问它们并对其进行描述。不会出现有人说“我不知道”这样的情况。此类决定至关重要，每个人都必须了解并遵循它们。其次它可以加快代码审查过程，在代码审查发现问题时您不必再写出错误的原因，而只需粘贴一个对应 ADR 条目的链接，而不用解释在各种场景下为什么这样做，要怎么修改这些细节。

## 总结

每个系统都有架构。 问题是我们要塑造系统的架构还是任其自我演进？任其自我演进很可能会导致项目出大问题。

架构实施是每个团队成员（不仅仅是架构师）的责任，至关重要。我们需要确保实施过程遵循架构，以上技术可以显著促进和改善架构实施过程，同时将我们的系统质量保持在适当的水平。
