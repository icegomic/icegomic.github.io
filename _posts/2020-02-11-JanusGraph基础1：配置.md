---
layout:     post
title:      JanusGraph基础1：配置
subtitle:   
date:       2020-02-11
author:     IceGomic
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - 知识图谱
    - 图数据库
    - NLP
---


> 本文首次发布于 [IceGomic Blog](http://icegomic.github.io), 作者 [@IceGomic](http://github.com/icegomic) ,转载请保留原文链接.

JanusGraph图形数据库集群由一个或多个JanusGraph实例组成。 要打开JanusGraph实例，必须提供一个配置，该配置指定应如何设置JanusGraph。

JanusGraph配置可以指定JanusGraph使用哪些组件，控制JanusGraph部署的所有可选内容，并提供许多调整选项以使得JanusGraph集群达到最佳性能。


JanusGraph配置至少需要定义一个持久引擎，JanusGraph会将其用作存储后端。 [存储后端页面](https://docs.janusgraph.org/storage-backend/)列出了所有可支持的持久引擎以及如何分别配置它们。 如果需要高级图形查询支持（例如，全文搜索，地理位置信息搜索或范围查询），则必须配置其他索引后端。 有关详细信息，请参见[索引后端](https://docs.janusgraph.org/index-backend/search-predicates/) 如果需要考虑查询性能，则应启用缓存。 缓存配置和调整在[JanusGraph缓存](https://docs.janusgraph.org/basics/configuration/#caching)中进行了描述。
## 配置的栗子🌰

以下是一些示例配置文件，以演示如何配置最常用的存储后端，索引系统和性能组件。这仅涵盖了可用配置选项的一小部分。
参考 [配置参考](https://docs.janusgraph.org/basics/configuration-reference/) 以了解全部的可选项。

### Cassandra+Elasticsearch

将JanusGraph设置为使用本地运行的Cassandra持久引擎和远程ES索引系统：
```
storage.backend=cql
storage.hostname=localhost

index.search.backend=elasticsearch
index.search.hostname=100.100.101.1, 100.100.101.2
index.search.elasticsearch.client-only=true
```

### HBase+Caching
将JanusGraph设置为使用远程运行的HBase持久引擎，并使用JanusGraph的缓存组件以提高性能。
```

storage.backend=hbase
storage.hostname=100.100.101.1
storage.port=2181

cache.db-cache = true
cache.db-cache-clean-wait = 20
cache.db-cache-time = 180000
cache.db-cache-size = 0.5
```
### BerkeleyDB
将JanusGraph设置为用BerkeleyDB作为嵌入式持久性引擎，并将Elasticsearch用作嵌入式索引系统。
```
storage.backend=berkeleyje
storage.directory=/tmp/graph

index.search.backend=elasticsearch
index.search.directory=/tmp/searchindex
index.search.elasticsearch.client-only=false
index.search.elasticsearch.local-mode=true
```
[配置参考](https://docs.janusgraph.org/basics/configuration-reference/)详细描述了所有这些配置选项。 JanusGraph发行版的conf目录包含其他配置示例。
### 更多示例

There are several example configuration files in the `conf/` directory that can be used to get started with JanusGraph quickly. Paths to these files can be passed to `JanusGraphFactory.open(...)` as shown below:
`conf /`目录中有几个示例的配置文件，可使用他们快速开始JanusGraph。 这些文件的路径可以配置到`JanusGraphFactory.open(...)`，如下所示：

```
// 使用默认的配置文件连接到本地 Cassandra 
graph = JanusGraphFactory.open("conf/janusgraph-cql.properties")
// 使用默认的配置文件连接到本地 HBase
graph = JanusGraphFactory.open("conf/janusgraph-hbase.properties")
```
### 使用配置

根据不同的实例化模式有几种不同的方式用于配置JanusGraph。

### JanusGraphFactory
#### Gremlin Console
JanusGraph发行版包含一个命令行工具Gremlin Console，可以用来与JanusGraph进行交互，并且很容易上手。 调用bin/gremlin.sh（Unix / Linux）或bin/gremlin.bat（Windows）来启动控制台，然后使用存储在可访问的属性配置文件中的默认配置打开JanusGraph：
```
graph = JanusGraphFactory.open('path/to/configuration.properties')
```

####  JanusGraph Embedded
我们还可以在基于JVM的用户应用程序中通过调用JanusGraphFactory类来启动一个嵌入的JanusGraph图实例。在这样的情况下，JanusGraph是这个应用程序的一部分，而且这个应用程序可以通过公共API接口方便的调用JanusGraph。

#### Short Codes

如果先前已经配置了JanusGraph图集群和/或仅需要定义存储后端，则JanusGraphFactory可以接收以冒号分隔的字符串表示形式，用来传递存储后端名称和主机名或目录，如下所示。
```
graph = JanusGraphFactory.open('cql:localhost')
graph = JanusGraphFactory.open('berkeleyje:/tmp/graph')
```
### JanusGraph Server

JanusGraph本身也可以认为是一组没有执行线程的jar文件。 有两种连接和使用JanusGraph数据库的基本模式：
1.  一个程序的可执行线程可以通过调用嵌入的JanusGraph方式来使用JanusGraph。

2.  JanusGraph打包了一个可长期运行的服务器进程，启动该进程后，它允许远程客户端或运行在单独程序中的逻辑调用JanusGraph。 这个长期运行的服务器进程称为** JanusGraph Server **。

JanusGraph Server 使用[Apache TinkerPop](https://tinkerpop.apache.org/) 的 [Gremlin Server](https://tinkerpop.apache.org/docs/3.4.4/reference/#gremlin-server) 堆栈来处理客户端请求。JanusGraph提供了开箱即用的配置，可快速启动JanusGraph Server，但可以更改配置以提供更多功能。


通过位于JanusGraph发行版`./conf/gremlin-server`目录中的JanusGraph Server yaml配置文件来配置JanusGraph Server。 要使用JanusGraph Server配置一个图实例（JanusGraph`），JanusGraph Server配置文件需要以下设置：
```
...
graphs: {
  graph: conf/janusgraph-berkeleyje.properties
}
scriptEngines: {
  gremlin-groovy: {
    plugins: { org.janusgraph.graphdb.tinkerpop.plugin.JanusGraphGremlinPlugin: {},
               org.apache.tinkerpop.gremlin.server.jsr223.GremlinServerGremlinPlugin: {},
               org.apache.tinkerpop.gremlin.tinkergraph.jsr223.TinkerGraphGremlinPlugin: {},
               org.apache.tinkerpop.gremlin.jsr223.ImportGremlinPlugin: {classImports: [java.lang.Math], methodImports: [java.lang.Math#*]},
               org.apache.tinkerpop.gremlin.jsr223.ScriptFileGremlinPlugin: {files: [scripts/empty-sample.groovy]}}}}
...

```
`graphs`这一个条目中定义了‘捆绑物bindings’来明确`JanusGraph`的配置。在上面的栗子中，把 `graph`绑定到了`conf/janusgraph-berkeleyje.properties`这个配置文件上。`plugins`这个条目启用了JanusGraph Gremlin Plugin，这样可以自动导入JanusGraph类，以便可以在远程提交的脚本中引用它们。

想了解更多关于JanusGraph Server 的配置和使用方法可参见 [JanusGraph Server](https://docs.janusgraph.org/basics/server/).

#### 服务器分配

JanusGraph zip文件包含一个快速启动服务器组件，该组件有助于简化和快速启动Gremlin Server和JanusGraph。 调用`bin/janusgraph.sh start`来启动Gremlin Server,并使用Cassandra和Elasticsearch做后端。
```
- 注意事项
为了安全起见， Elasticsearch 和 `janusgraph.sh` 应该使用非root账户启动。
```
## 全局配置

JanusGraph的配置选项区分为本地配置选项和全局配置选项。 本地配置选项适用于单个JanusGraph实例。 全局配置选项适用于集群中的所有实例。 更具体地说，JanusGraph为配置选项区分了以下五个级别：

*   **LOCAL**: 这些选项仅适用于单个JanusGraph实例，并在初始化JanusGraph实例时提供的配置文件中指定。

*   **MASKABLE**: 本地配置文件可以为单个JanusGraph实例覆盖这些配置选项。 如果本地配置文件未指定该选项，则从全局JanusGraph群集配置中读取其值。

*   **GLOBAL**: 这些选项始终从集群配置中读取，不能基于实例进行覆盖。

*   **GLOBAL_OFFLINE**: 类似于 *GLOBAL*, 但是可以更改，更改这些选项需要重新启动群集，以确保整个群集中的值相同。


*   **FIXED**: 类似于 *GLOBAL*, 但是一旦JanusGraph集群初始化，就无法更改该值。



当第一个集群中的JanusGraph实例启动时，将从提供的本地配置文件中初始化全局配置选项。 随后，通过JanusGraph的管理API更改全局配置选项。 要访问管理API，请在打开的JanusGraph实例句柄`g`上调用`g.getManagementSystem() `。 例如，要更改JanusGraph群集上的默认缓存行为可以使用下面的栗子：
```
mgmt = graph.openManagement()
mgmt.get('cache.db-cache')
// Prints the current config setting 输出当前配置
mgmt.set('cache.db-cache', true)
// Changes option 修改配置
mgmt.get('cache.db-cache')
// Prints 'true'输出 
mgmt.commit()
// Changes take effect
```
### 修改离线配置


更改配置选项不会影响正在运行的实例，仅会在新启动的实例上生效。 更改 *GLOBAL_OFFLINE* 配置选项需要重新启动群集，以便更改对所有实例立即生效。 要更改 *GLOBAL_OFFLINE* 选项，请按照以下步骤操作：
*   只保留一个集群中的实例，其他所有实例全部关掉
*   连接到该实例
*   确认所有事务都已经关闭
*   确认不会有新的事务启动 (i.e. 集群必须是下线模式)
*   启动管理API
*   修改配置选项
*   提交配置，该调用会自动关闭实例
*   重启全部实例

要了解全部配置详情请参考 [配置说明](https://docs.janusgraph.org/basics/configuration-reference/) 包括配置选项和选项的配置范围。
