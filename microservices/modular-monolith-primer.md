本文翻译自 Kamil Grzybek 的博客文章，原文地址：
> https://www.kamilgrzybek.com/design/modular-monolith-primer/

![Modular Monolith](https://github.com/hotjk/translation/blob/master/microservices/mm/Modular_Monolith_a_Primer-825x510.jpg)
 
# 模块化单体：基础

这篇文章是关于模块化单体架构的系列文章的一部分：

1. 模块化单体：基础
2. [模块化单体：架构驱动因素](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-architectural-drivers.md)
3. [模块化单体：架构实施](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-architecture-enforcement.md)
4. [模块化单体：集成风格](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-integration-styles.md)
5. [模块化单体：以领域为中心的设计](https://github.com/hotjk/translation/blob/master/microservices/modular-monolith-domain-centric-design.md)


## 介绍

在系统架构领域，流行了很多年的微服务（microservice）架构仍然是热门的话题。基于云的解决方案和使用类如 Kubernetes 的容器化开发部署模式的普及进一步加重了微服务的流行。

从开发社区、各企业以及程序员之间的交流中都可以看到，新的项目大多都采用了微服务架构，甚至一些老的系统也在做微服务架构的迁移。

这次我想谈及的是对应微服务架构的模块化单体（Modular Monolith）架构，为什么要写这个主题，因为我觉得目前行业已经很大程度的在误用微服务架构了。大家认为微服务是解决单体应用程序所有问题的良药，而不再关心驱动架构的因素。你只要参与过由多个部署单元组成的分布式系统的开发，你就知道每种架构都有其优点和缺点，微服务也不例外。他们解决一些问题的同时也带来了新的问题。

我决定编写这个关于模块化单体架构的系列文章主要出于以下原因：首先，我想驳斥那种单体架构不能写出“高级”应用的错误观点；其次，我想打消对模块化单体架构的定义和形式的怀疑，很多人并不真正理解模块化单体架构。最后，本系列文章也是我之前一篇文章 [使用 DDD 来实现模块化单体应用](https://github.com/kgrzybek/modular-monolith-with-ddd) 的扩展和补充，我几个月前在 GitHub 上分享了这篇文章很受欢迎（一个月获得 1000 颗星）。

在本系列文章的第一篇，我将重点介绍模块化单体（Modular Monolith）架构的定义。

## 什么是模块化单体？

当我谈论或者撰写技术和业务问题时，我总是力求准确，尤其是在涉及架构时。我相信一个清晰连贯的定义非常重要。这就是为什么我想明确定义模块化单体架构以及我如何理解该架构。让我们从简单的概念开始，什么是单体（Monolith）？

### 单体 Monolith

[维基百科](https://en.wikipedia.org/wiki/Monolithic_architecture) 从建筑构造视角而不是计算机科学视角对单体架构的定义如下：

> 整体式建筑（单体架构）描述了一种使用整块材料（比如岩石）进行雕刻、铸造或挖掘来建造建筑的方式。

对应于计算机科学领域，建筑就是计算机系统，而材料就是我们的可执行代码。所以在单体架构中，我们的系统只有一份不可拆分的可执行代码。

让我们再看两个技术名词：第一个是 [单体系统（Monolith System）](https://en.wikipedia.org/wiki/Monolithic_system)：

> 在单体架构软件系统中，各维度的功能（例如数据输入和输出、数据处理、错误处理和用户界面）都交织在一起，而不是独立的组件。

第二个技术名词是 [单体架构（Monolithic Architecture）](https://whatis.techtarget.com/definition/monolithic-architecture)：

> 单体架构是软件程序设计的传统模型。单体意味着各功能组件全部组合在一起。 单体软件是自成一体的，程序的组件是相互关联和相互依赖的，而不是像模块化软件程序那样松散耦合的。

通过 Google 上面两个技术名词，你可以发现他们有两个共同点。首先，他们都表达了此类架构的所有部分都在一个部署单元，我同意这一点。

第二个共同点是，他们假设这种架构缺乏模块化，我绝对不同意这一点。“功能交织在一起，而不是独立的组件”，“程序的组件是相互关联和相互依赖的，而不是像模块化软件程序那样松散耦合的”，这些对架构的描述是非常负面地。事实可能是这样，也可能不是这样，单体架构也可以模块化。

### 模块化 Modularization

[词典](https://dictionary.cambridge.org/dictionary/english/modular) 上对模块化的（modular）的定义：

> 由单独的部分组成，当组合时形成一个完整的整体
> 由一组单独的部分组成，这些部分可以连接在一起形成一个更大的物体

对模块化（Modularization）的定义是：

> 使用分离部分的方式来设计或生产某物

这是一个通用的定义，对于编程世界来说还不够精确。让我们看一个更具体的名词 [模块化编程（Modular programming）](https://en.wikipedia.org/wiki/Modular_programming) 的定义：

> 模块化编程是一种软件设计技术，它强调将程序的功能分成独立的、可互换的模块，这样每个模块都包含执行某一方面功能所需的一切。模块的接口表达了模块提供的能力和需要的输入。模块的接口可以被其他模块感知。而模块的实现包含了与接口中声明的能力相对应的工作代码。

看起来，为了实现模块化架构，我们必须构建模块，并且模块还需要满足以下几个关键点：

- 模块必须是独立的和可互换的
- 模块必须具备提供对应功能所需的一切
- 模块必须有良好的接口定义
  
让我们详细分析下这几点的意思。

#### 模块必须是独立的和可互换的

模块必须是独立的。当然，模块不可能完全没有依赖，那意味着模块无法与其他模块集成。模块总是会依赖某些东西，但应该将依赖关系保持在最低限度内，原则就是：[松耦合，强内聚](http://www.kamilgrzybek.com/design/grasp-explained/)。

在下图左侧我们看到一个模块有很多依赖项，这样的模块就不是独立的。右边的情况恰好相反，模块只包含最少的依赖，他们更加松散，模块也更独立。

![模块独立性](https://github.com/hotjk/translation/blob/master/microservices/mm/Module_independence-768x315.jpg?raw=true)

然而，依赖的数量只是衡量模块独立程度的一个标准。第二个衡量标准是依赖的强度。换句话说，模块之间是经常互相调用或者调用方式多种多样，还是偶尔调用并且仅使用一种或有限的几种方法调用方式？

![强依赖/弱依赖](https://github.com/hotjk/translation/blob/master/microservices/mm/Module_independence_strongweak-768x315.jpg?raw=true)

如果是第一种情况，我们可能错误地定义了模块的边界，如果两个模块密切相关，我们应该合并它们：

![合并模块](https://github.com/hotjk/translation/blob/master/microservices/mm/Module_indpendence_merge-768x315.jpg?raw=true)

最后一个影响模块独立性的属性是它所依赖的模块的变更频率。您猜的没错，它们变更频率越低，模块就越独立。反之，如果依赖项变更频繁，我们必须经常随着一起变更我们的模块，模块也就失去了独立性：

![模块稳定性](https://github.com/hotjk/translation/blob/master/microservices/mm/Module_independence_stability-768x355.jpg?raw=true)

综上所述，模块的独立性由三个主要因素决定：

- 依赖项的数量
- 依赖强度
- 模块依赖的其他模块的稳定性

#### 模块必须具备提供对应功能所需的一切

从不同的视角出发，定义模块的方式也各不相同。最传统的方式是按系统的逻辑层级来定义模块，比如 GUI 模块、应用逻辑模块、数据库访问模块。没错，在技术视角这些也是模块，但它们只是技术功能模块，而不是业务功能模块。

这种从技术视角定义的模块，只有使用的技术发生变化才会导致单一模块发生修改：

![技术模块和技术变更](https://github.com/hotjk/translation/blob/master/microservices/mm/TechnicalModules_technicalChange-768x355.jpg?raw=true)

而添加或者修改业务功能通常要遍历所有的业务层级，并导致所有技术模块都要修改：

![技术模块下增加/修改业务功能](https://github.com/hotjk/translation/blob/master/microservices/mm/TechnicalModule_FeatureChange-768x316.jpg?raw=true)

技术变更频繁还是业务变更频繁？显而易见，肯定是业务变更频繁。我们很少更换数据库访问层、日志库或者 GUI 框架。因此，模块化单体架构中的模块是一个能够完全提供一组特定业务功能的业务模块。这种设计称为“垂直拆分”，将业务垂直拆分为模块：

![业务模块垂直拆分](https://github.com/hotjk/translation/blob/master/microservices/mm/BusinessModules_VerticalSlices-1-768x302.jpg?raw=true)

这样，某个频繁变更的业务只会影响一个模块，模块变得更加独立、自治。

#### 模块必须有良好的接口定义

模块化的最后一个关键点是定义良好的接口。没有接口做合约，谈何模块化架构：

![没有接口合约的模块](https://github.com/hotjk/translation/blob/master/microservices/mm/Modules_without_contract-768x411.jpg?raw=true)

合约是模块对外提供能力的能力，至关重要。 它是模块的“入口点”。良好的合约应该是明确的，并且只包含客户需要的内容。我们应该保持合约稳定（模块的客户依赖模块的合约）并将其他所有内容隐藏在它后面（[封装](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming))）：

![有接口合约的模块](https://github.com/hotjk/translation/blob/master/microservices/mm/Modules_with_contract-768x411.jpg?raw=true)

如上图所示，模块的接口合约可以是多种形式。有时是同步调用的接口（例如公共方法或 REST 服务），有时是异步通信的事件。模块对外提供的所有能力都将成为模块的公共 API。封装是模块化的关键。

## 总结

1. 单体系统是指只有一个部署单元的系统。
2. 单体架构并不意味着系统设计不佳或者没有模块化。
3. 模块化单体架构是以模块化方式设计的单体系统的明确名称。
4. 为了实现高水平的模块化，每个模块必须是独立的，具有提供特定功能所需的一切（按业务领域分离），封装良好并具有明确定义的接口合约。

在下一篇文章中，我将讨论模块化单体架构与微服务相比的优缺点。
