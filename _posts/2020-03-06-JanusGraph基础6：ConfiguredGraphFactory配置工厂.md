---
layout:     post
title:      JanusGraph基础6：ConfiguredGraphFactory配置工厂
subtitle:   
date:       2020-03-06
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

可以将JanusGraph Server配置为使用`ConfiguredGraphFactory`。 `ConfiguredGraphFactory` 是对图形的访问点，类似于`JanusGraphFactory`。 这些图工厂提供了用于动态管理服务器上托管的图的方法。
## 总览
`JanusGraphFactory` 类提供了一个图的访问点，每次你访问图的时候它会创建一个配置好的对象。

`ConfiguredGraphFactory`为您的图形提供了访问点，您之前已经使用 `ConfigurationManagementGraph`为其创建了配置。 它还提供了一个访问点来管理图配置。

`ConfigurationManagementGraph`允许您管理图配置。

`JanusGraphManager`是一个内部服务器组件，可以跟踪图引用，前提是您的图形已配置为使用它。
## ConfiguredGraphFactory 与 JanusGraphFactory

这两个图形工厂之间存在重要区别：
1.  仅当您将服务器配置为在服务启动时使用ConfigurationManagementGraph APIs时，才可以使用`ConfiguredGraphFactory`。
使用 `ConfiguredGraphFactory` 有很多好处:

1.  与`JanusGraphFactory`相对，您只需提供一个 `String` 即可访问您的图形。`JanusGraphFactory`则需要每次打开一个图时，指定有关要使用的后端的信息。
2.   如果您的ConfigurationManagementGraph配置有分布式存储后端，则您的图配置可用于集群中的所有JanusGraph节点。
## ConfiguredGraphFactory 是怎样工作的?

在下列两种情况下，`ConfiguredGraphFactory` 提供了一个接入点:

1.  您已经使用 `ConfigurationManagementGraph#createConfiguration为某个图对象创建了配置。 在这种情况下，将使用先前为此图创建的配置来打开该图。
2.  您已经使用 `ConfigurationManagementGraph#createTemplateConfiguration`创建了模板配置。 在这种情况下，会复制存储在模板配置中的所有属性并附加相关的graphName属性，为要创建的图形创建配置，然后根据该特定配置打开图形。

## 访问Graphs
  
您可以使用 `ConfiguredGraphFactory.create("graphName")`或`ConfiguredGraphFactory.open("graphName")`。 如果想了解有关这两个选项之间差异的更多信息，可以阅读下面有关`ConfigurationManagementGraph`的部分。

您还可以通过使用绑定来访问图。 在[图形和遍历绑定](https://docs.janusgraph.org/master/basics/configured-graph-factory/#graph-and-traversal-bindings)中阅读有关此内容的更多信息
## 列出Graphs

`ConfiguredGraphFactory.getGraphNames()` 会返回一个graph名称的集合，这个集合是你调用`ConfigurationManagementGraph` APIs 创建和配置的图名称集合。 

另一方面，`JanusGraphFactory.getGraphNames()` 将返回一个graph名称的集合，这些是你实例化的图名称，并且引用存储在JanusGraphManager中。
## 删除图

`ConfiguredGraphFactory.drop("graphName")` 将会删除图数据库，删除存储后端和索引后端上面的所有数据。 图形可以打开或关闭（执行删除操作时会先关闭）。 此外，这还将删除`ConfigurationManagementGraph`中的任何现有图配置。

> *重要提示*
这是不可逆的操作，将删除所有图和索引数据。

> *重要提示*
为了确保所有图形表示形式在集群中的所有JanusGraph节点上都是一致的，假设已将每个节点正确配置为使用`JanusGraphManager`，则会从群集中每个节点上的`JanusGraphManager`图缓存中删除该图。 在此[此处](https://docs.janusgraph.org/master/basics/multi-node/#graph-reference-consistency)了解有关此功能以及如何配置服务以使用该功能的更多信息。
## 为ConfiguredGraphFactory 配置 JanusGraph Server  

为了能够使用`ConfiguredGraphFactory`，必须将服务器配置为使用`ConfigurationManagementGraph` API。 为此，您必须在服务器的YAML的“图”中注入一个名为"ConfigurationManagementGraph"的图变量。 例如：
```
graphManager: org.janusgraph.graphdb.management.JanusGraphManager
graphs: {
  ConfigurationManagementGraph: conf/JanusGraph-configurationmanagement.properties
}
```
在此示例中，将使用存储在conf/JanusGraph-configurationmanagement.properties中的属性来配置我们的ConfigurationManagementGraph图，例如，如下所示：
```
gremlin.graph=org.janusgraph.core.ConfiguredGraphFactory
storage.backend=cql
graph.graphname=ConfigurationManagementGraph
storage.hostname=127.0.0.1
```
假设GremlinServer成功启动并且ConfigurationManagementGraph成功实例化，则ConfigurationManagementGraph Singleton上所有可用的API也会作用于该图。 此外，该图将用于访问使用ConfiguredGraphFactory创建/打开图的配置。
> *重要提示*
JanusGraph发行版中包含的pom.xml将此依赖关系列为可选，但是ConfiguredGraphFactory使用JanusGraphManager，这需要在org.apache.tinkerpop:gremlin-server上声明依赖。 因此，如果遇到NoClassDefFoundError错误，请确保根据此消息进行更新。
## ConfigurationManagementGraph
ConfigurationManagementGraph是一个Singleton(单例模式)，允许您创建/更新/删除可用于使用ConfiguredGraphFactory访问图形的配置。 有关配置服务器以启用这些API的信息，请参见上文。

> *重要提示*
ConfiguredGraphFactory提供了一个访问点来管理由ConfigurationManagementGraph管理的图形配置，因此，您可以对相应的ConfiguredGraphFactory静态方法进行操作，而不是对Singleton本身进行操作。 例如，您可以使用ConfiguredGraphFactory.removeTemplateConfiguration()来代替ConfiguredGraphFactory.getInstance().removeTemplateConfiguration()。
### Graph 配置
ConfigurationManagementGraph单例模式允许您创建用于打开特定图的配置，并由graph.graphname属性引用。 例如：
 ```
map = new HashMap<String, Object>();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
map.put("graph.graphname", "graph1");
ConfiguredGraphFactory.createConfiguration(new MapConfiguration(map));
```
接下来你可以使用下面这句配置，从而可以在任何JanusGraph 节点访问该图:
```
ConfiguredGraphFactory.open("graph1");
```
### 模板配置

ConfigurationManagementGraph还允许您创建一个模板配置，通过该模板配置，您可以使用同一配置模板创建许多图。 例如：  
```
map = new HashMap<String, Object>();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
ConfiguredGraphFactory.createTemplateConfiguration(new MapConfiguration(map));
```
经过上面的操作之后 你可以使用模板配置来创建图了:
```
ConfiguredGraphFactory.create("graph2");
```

此方法将首先通过复制与模板配置关联的所有属性并将其存储在此特定图形的配置中，为“ graph2”创建一个新配置。 这意味着以后可以通过以下操作在任何JanusGraph节点上访问此图：
```
ConfiguredGraphFactory.open("graph2");
```
### 更新配置信息
与JanusGraphFactory和ConfiguredGraphFactory的所有交互都与通过定义Jan.GraphManager属性的配置进行交互的JanusGraphManager进行交互，JanusGraphManager跟踪在给定JVM上创建的图形引用。 `JanusGraphFactory` 和 `ConfiguredGraphFactory` 与配置信息的所有交互都将在给定的JVM中进行。 该配置通过使用 `JanusGraphManager` 来跟踪图的引用情况，并定义了`graph.graphname` 属性。你可以将其视为图缓存。 为此原因：
> *重要提示*
假设使用JanusGraphManager已正确配置每个节点，则对图形配置的任何更新都会导致JanusGraph群集中每个节点上的图形缓存被逐出相关图形。 在此[此处](https://docs.janusgraph.org/master/basics/multi-node/#graph-reference-consistency)了解有关此功能以及如何配置服务器以使用该功能的更多信息。

由于使用模板配置创建的图首先使用复制和创建方法为该图创建配置，所以这意味着：

> *重要提示*
在下面两种情况下，使用模板配置创建的特定图形的任何更新不能保证生效：1\.删除了相关配置：`ConfiguredGraphFactory.removeConfiguration("graph2");` 2\. 使用模板配置重新创建该图： `ConfiguredGraphFactory.create("graph2");`
### 更新范例

1) 我们将Cassandra数据迁移到了新服务器上，换了新IP地址：
```
map = new HashMap();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
map.put("graph.graphname", "graph1");
ConfiguredGraphFactory.createConfiguration(new
MapConfiguration(map));

g1 = ConfiguredGraphFactory.open("graph1");

// Update configuration
map = new HashMap();
map.put("storage.hostname", "10.0.0.1");
ConfiguredGraphFactory.updateConfiguration("graph1",
map);

// We are now guaranteed to use the updated configuration
g1 = ConfiguredGraphFactory.open("graph1");
```
2) 我们在设置中添加了Elasticsearch节点：
```
map = new HashMap();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
map.put("graph.graphname", "graph1");
ConfiguredGraphFactory.createConfiguration(new
MapConfiguration(map));

g1 = ConfiguredGraphFactory.open("graph1");

// Update configuration
map = new HashMap();
map.put("index.search.backend", "elasticsearch");
map.put("index.search.hostname", "127.0.0.1");
map.put("index.search.elasticsearch.transport-scheme", "http");
ConfiguredGraphFactory.updateConfiguration("graph1",
map);

// We are now guaranteed to use the updated configuration
g1 = ConfiguredGraphFactory.open("graph1");
```
3) 更新图形配置，该配置使用最新的模板配置创建：
```
map = new HashMap();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
ConfiguredGraphFactory.createTemplateConfiguration(new
MapConfiguration(map));

g1 = ConfiguredGraphFactory.create("graph1");

// Update template configuration
map = new HashMap();
map.put("index.search.backend", "elasticsearch");
map.put("index.search.hostname", "127.0.0.1");
map.put("index.search.elasticsearch.transport-scheme", "http");
ConfiguredGraphFactory.updateTemplateConfiguration(new
MapConfiguration(map));

// Remove Configuration
ConfiguredGraphFactory.removeConfiguration("graph1");

// Recreate
ConfiguredGraphFactory.create("graph1");
// Now this graph's configuration is guaranteed to be updated
```
## JanusGraphManager
JanusGraphManager是遵循TinkerPop graphManager规范的Singleton（单例模式）。
特别是，JanusGraphManager提供：
1. 一种协调机制，在给定的JanusGraph节点上实例化图形引用
2.图引用跟踪器（或缓存）
使用graph.graphname属性创建的任何图形都将通过JanusGraphManager，因此它们以协调的方式实例化。 图形引用也将放置在有问题的JVM上的图形缓存中。

因此，使用已在有关JVM上实例化的使用graph.graphname属性打开的任何有问题的图形都将从图形缓存中检索。

这就是为什么对配置进行更新需要几个步骤来保证正确性的原因。
### 如何使用 JanusGraphManager

这是一个你可以使用的新配置选项。它定义了如何访问一个图。包含此属性的所有配置都将导致通过JanusGraphManager发生图实例化（上面说明了过程）。

为了向后兼容，任何不提供此参数的图，但是在服务器上运行的图都将在.yaml文件中的图对象中开始，这些图形将通过JanusGraphManager绑定，该JanusGraphManager用为该图形提供的键表示。 例如，如果您的.yaml图形对象应该是：
```
graphManager: org.janusgraph.graphdb.management.JanusGraphManager
graphs {
  graph1: conf/graph1.properties,
  graph2: conf/graph2.properties
}
```

但是conf/graph1.properties和conf/graph2.properties不包含属性graph.graphname，那么这些图将存储在JanusGraphManager中，并因此分别以graph1和graph2绑定在您的gremlin脚本执行中。

 ### 重要提示
为了方便起见，如果用于打开图形的配置指定了graph.graphname，但未指定后端的存储目录，表名或键空间名，则相关参数将自动设置为graph.graphname的值。 但是，如果提供这些参数之一，则该值将始终优先。 而且，如果您都不提供，则它们默认为配置选项的默认值。
一种特殊情况是storage.root配置选项。 这是一个新的配置选项，用于指定目录的基础，该目录将用于所有需要本地存储目录访问的后端。 如果提供此参数，则还必须提供graph.graphname属性，并且绝对存储路径需要和加到storage.root属性值之后的graph.graphname属性值保持一致。
以下是一些示例用例：

1) 为我的Cassandra后端创建一个模板配置，这样，使用此配置创建的每个图形都将获得一个唯一的键空间，该键空间等效于提供给工厂的String <graphName>：
```
map = new HashMap();
map.put("storage.backend", "cql");
map.put("storage.hostname", "127.0.0.1");
ConfiguredGraphFactory.createTemplateConfiguration(new
MapConfiguration(map));

g1 = ConfiguredGraphFactory.create("graph1"); //keyspace === graph1
g2 = ConfiguredGraphFactory.create("graph2"); //keyspace === graph2
g3 = ConfiguredGraphFactory.create("graph3"); //keyspace === graph3
```
2) 为我的BerkeleyJE后端创建模板配置，以使使用此配置创建的每个图形都具有一个等效于“ <storage.root>/<graph.graphname>”的唯一存储目录：
```
map = new HashMap();
map.put("storage.backend", "berkeleyje");
map.put("storage.root", "/data/graphs");
ConfiguredGraphFactory.createTemplateConfiguration(new
MapConfiguration(map));

g1 = ConfiguredGraphFactory.create("graph1"); //storage directory === /data/graphs/graph1
g2 = ConfiguredGraphFactory.create("graph2"); //storage directory === /data/graphs/graph2
g3 = ConfiguredGraphFactory.create("graph3"); //storage directory === /data/graphs/graph3
```
## 图的遍历绑定

使用ConfiguredGraphFactory创建的图通过"graph.graphname"属性绑定到Gremlin服务上的执行器上连接，图的遍历引用通过"`<graphname>_traversal`"绑定到连接。 这意味着，在首次创建/打开图形之后，在与服务器的后续连接上，可以通过“`<graphname>”和“ <graphname> _traversal”属性访问图形和遍历引用。

在此[此处](https://docs.janusgraph.org/master/basics/multi-node/#dynamic-graph-and-traversal-bindings)了解有关此功能以及如何配置服务器以使用该功能的更多信息。
> *重要提示*
如果使用Gremlin控制台和**会话**连接到远程Gremlin服务器，则必须重新连接到服务器以绑定变量。 对于任何会话式WebSocket连接也是如此。
> *重要提示*
JanusGraphManager每20秒重新绑定存储在ConfigurationManagementGraph上的每个图（或为其创建配置的图）。 这意味着使用ConfiguredGraphicFactory创建的图形的图形绑定和遍历绑定将在所有JanusGraph节点上可用，最多滞后20秒。 这也意味着在服务器重新启动后，绑定在节点上仍然可用。
### 绑定例子
```
gremlin> :remote connect tinkerpop.server conf/remote.yaml
==>Configured localhost/127.0.0.1:8182
gremlin> :remote console
==>All scripts will now be sent to Gremlin Server - [localhost/127.0.0.1:8182] - type ':remote console' to return to local mode
gremlin> ConfiguredGraphFactory.open("graph1")
==>standardjanusgraph[cassandrathrift:[127.0.0.1]]
gremlin> graph1
==>standardjanusgraph[cassandrathrift:[127.0.0.1]]
gremlin> graph1_traversal
==>graphtraversalsource[standardjanusgraph[cassandrathrift:[127.0.0.1]], standard]
```
## 例子  
在创建“配置的图工厂”模板时，建议使用会话连接。 如果未使用会话连接，则必须将“已配置图形工厂模板”创建语句写在一行里发送到服务器，其中的配置项使用分号分隔。 可以在[连接到Gremlin服务器](https://docs.janusgraph.org/master/connecting/)中找到有关会话的详细信息。
```
gremlin> :remote connect tinkerpop.server conf/remote.yaml session
==>Configured localhost/127.0.0.1:8182

gremlin> :remote console
==>All scripts will now be sent to Gremlin Server - [localhost:8182]-[5206cdde-b231-41fa-9e6c-69feac0fe2b2] - type ':remote console' to return to local mode

gremlin> ConfiguredGraphFactory.open("graph");
Please create configuration for this graph using the
ConfigurationManagementGraph API.

gremlin> ConfiguredGraphFactory.create("graph");
Please create a template Configuration using the
ConfigurationManagementGraph API.

gremlin> map = new HashMap();
gremlin> map.put("storage.backend", "cql");
gremlin> map.put("storage.hostname", "127.0.0.1");
gremlin> map.put("GraphName", "graph1");
gremlin> ConfiguredGraphFactory.createConfiguration(new MapConfiguration(map));
Please include in your configuration the property "graph.graphname".

gremlin> map = new HashMap();
gremlin> map.put("storage.backend", "cql");
gremlin> map.put("storage.hostname", "127.0.0.1");
gremlin> map.put("graph.graphname", "graph1");
gremlin> ConfiguredGraphFactory.createConfiguration(new MapConfiguration(map));
==>null

gremlin> ConfiguredGraphFactory.open("graph1").vertices();

gremlin> map = new HashMap(); map.put("storage.backend",
"cql"); map.put("storage.hostname", "127.0.0.1");
gremlin> map.put("graph.graphname", "graph1");
gremlin> ConfiguredGraphFactory.createTemplateConfiguration(new MapConfiguration(map));
Your template configuration may not contain the property
"graph.graphname".

gremlin> map = new HashMap();
gremlin> map.put("storage.backend",
"cql"); map.put("storage.hostname", "127.0.0.1");
gremlin> ConfiguredGraphFactory.createTemplateConfiguration(new MapConfiguration(map));
==>null

// Each graph is now acting in unique keyspaces equivalent to the
graphnames.
gremlin> g1 = ConfiguredGraphFactory.open("graph1");
gremlin> g2 = ConfiguredGraphFactory.create("graph2");
gremlin> g3 = ConfiguredGraphFactory.create("graph3");
gremlin> g2.addVertex();
gremlin> l = [];
gremlin> l << g1.vertices().size();
==>0
gremlin> l << g2.vertices().size();
==>1
gremlin> l << g3.vertices().size();
==>0

// After a graph is created, you must access it using .open()
gremlin> g2 = ConfiguredGraphFactory.create("graph2"); g2.vertices().size();
Configuration for graph "graph2" already exists.

gremlin> g2 = ConfiguredGraphFactory.open("graph2"); g2.vertices().size();
==>1
```
