---
layout:     post
title:      JanusGraph基础5：JanusGraph部署方案
subtitle:   
date:       2020-02-26
author:     IceGomic
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - 知识图谱
    - 图数据库
    - 人工智能
---


> 本文首次发布于 [IceGomic Blog](http://icegomic.github.io), 作者 [@IceGomic](http://github.com/icegomic) ,转载请保留原文链接.
JanusGraph使用[Gremlin Server](https://tinkerpop.apache.org/docs/3.4.5/reference/#gremlin-server)引擎作为服务器组件来处理和响应客户端查询。 当打包在JanusGraph中时，Gremlin Server称为JanusGraph Server。

JanusGraph提供了丰富的存储和索引后端选择，这为如何部署它提供了极大的灵活性。 本章介绍了一些可能的部署方案，以帮助解决这种灵活性带来的复杂性。

在讨论不同的部署方案之前，重要的是要了解JanusGraph本身和后端的所扮演的角色分工。 首先，应用程序仅直接与JanusGraph通信，主要是通过发送Gremlin命令来执行。 然后，JanusGraph与配置好的后端进行通信以执行接收到的查询命令。 当以JanusGraph Server的形式使用JanusGraph时，不需要区分主从服务器。 因此，应用程序可以连接到任何JanusGraph Server节点。 他们还可以使用负载均衡器来调度请求访问不通的节点。 JanusGraph Server实例本身不会直接相互通信，这使得在需要处理更多遍历时轻松扩展它们。
> 注意事项
本章介绍的方案只是如何部署JanusGraph的示例。 每个部署都需要考虑具体的用例和生产需求。

## 学习案例

这种情况是大多数用户刚开始使用JanusGraph时可能想要选择的情况。 它以最少的服务器数量提供可扩展性和容错能力。 JanusGraph Server与存储后端实例一起运行，还可以选择在每台服务器上同时与索引后端实例一起运行。

![image.png](https://upload-images.jianshu.io/upload_images/100763-09ff7375faf95039.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以通过简单地添加更多相同类型的服务器或将组件之一移动到专用服务器上来扩展这种设置。 后者描述了将部署方案转换为[高级方案](https://docs.janusgraph.org/master/basics/deployment/#advanced-scenario)的方法。

任何可扩展的存储后端都可用于此方案。 但是请注意，对于Scylla，[与其他服务托管在一起时需要一些配置](http://docs.scylladb.com/getting-started/scylla_in_a_shared_environment/) 就像在这种情况下一样。 当在这种情况下应该使用索引后端时，它也必须是可扩展的。

## 高级方案
高级方案是[入门方案](https://docs.janusgraph.org/master/basics/deployment/#getting-started-scenario)的演变。 现在，它们已经在不同的服务器上分离，存储后端以及索引后端已经不再和JanusGraph Server实例托管在一起。 在不同服务器上托管不同组件（JanusGraph Server，存储/索引后端）的优势在于，它们可以彼此独立地进行扩展和管理。 但是这需要维护更多服务器为代价，从而提供了更高的灵活性。
![image.png](https://upload-images.jianshu.io/upload_images/100763-812b27496eb4d59e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于此方案提供了不同组件的独立扩展方案，因此当然也可以使用可扩展的后端存储服务。
## 极简方案
也可以将JanusGraph Server和后端一起部署在一台服务器上。 对于测试目的或JanusGraph仅支持单个应用程序，该应用程序随后也可以在同一服务器上运行时，这尤其有吸引力。
![image.png](https://upload-images.jianshu.io/upload_images/100763-d75f510ae85fe38a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

与以前的方案相反，在此方案中使用不可扩展的后端是最好的。 内存后端可用于测试目的，伯克利数据库可用于生产，而Lucene可作为可选索引后端。
## 嵌入 JanusGraph
除了可以从应用程序连接到JanusGraph Server之外，还可以将JanusGraph作为库嵌入基于JVM的应用程序中。 尽管这减少了管理开销，但它使得无法独立于应用程序扩展JanusGraph。 嵌入式JanusGraph可以作为任何其他方案的变体来部署。 JanusGraph只是从服务器直接移到应用程序中，因为它现在仅用作库而不是独立的服务。