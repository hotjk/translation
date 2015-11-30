

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

不幸的是，似乎新一代的开发人员没有经历过