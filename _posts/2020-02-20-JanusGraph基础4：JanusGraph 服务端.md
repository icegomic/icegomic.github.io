---
layout:     post
title:      JanusGraph基础4：JanusGraph 服务端
subtitle:   
date:       2020-02-20
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

JanusGraph Server必须手动启动才能使用。 JanusGraph Server提供了对托管在其中的一个或多个JanusGraph实例远程执行Gremlin命令的方法。 本节将描述如何使用WebSocket配置，并讲解了如何配置JanusGraph Server来处理HTTP交互。 有关如何从不同语言连接到JanusGraph服务器的信息，请参阅[连接到JanusGraph](https://docs.janusgraph.org/master/connecting/)。
## 从这里开始

### 使用预打包的发行版

JanusGraph [release版本](https://github.com/JanusGraph/janusgraph/releases) 已经预先配置好了一个例子，开箱即用地运行JanusGraph Serve，可以利用Cassandra和Elasticsearch配置来让用户快速开始使用JanusGraph Server。 此配置默认为可以使用自定义子协议通过WebSocket连接到JanusGraph Server的客户端应用程序。 有许多使用不同语言开发的客户端可以支持该子协议。 使用WebSocket界面最熟悉的客户端是Gremlin Console。 快速入门包并非旨在代表生产安装，而是提供了一种使用JanusGraph Server进行开发，运行测试以及查看组件如何连接在一起的方法。 要使用此默认配置可以遵循下面的步骤：
*   从下面的页面下载一个最新版的`janusgraph-$VERSION.zip` 文件 ，这里 [Releases page](https://github.com/JanusGraph/janusgraph/releases)

*   解压文件并进入目录 `janusgraph-$VERSION`

*   运行 `bin/janusgraph.sh start`. 执行完这步之后，Gremlin Server就启动了，同时Cassandra/ES 作为独立进程也同时运行。安全起见，Elasticsearch 和 `janusgraph.sh` 要在非root账号下运行。 
```
$ bin/janusgraph.sh start
Forking Cassandra...
Running `nodetool statusthrift`.. OK (returned exit status 0 and printed string "running").
Forking Elasticsearch...
Connecting to Elasticsearch (127.0.0.1:9300)... OK (connected to 127.0.0.1:9300).
Forking Gremlin-Server...
Connecting to Gremlin-Server (127.0.0.1:8182)... OK (connected to 127.0.0.1:8182).
Run gremlin.sh to connect.
```
### 连接到 Gremlin Server
运行完 janusgraph.sh之后, Gremlin Server开始监听r WebSocket 连接请求.  使用Gremlin Console来测试是否能连接成功是最简单方式.

通过执行 bin/gremlin.sh来启动Gremlin Console，并使用:remote和:>命令将Gremlin语句发给Server：
```
$  bin/gremlin.sh
         \,,,/
         (o o)
-----oOOo-(3)-oOOo-----
plugin activated: tinkerpop.server
plugin activated: tinkerpop.hadoop
plugin activated: tinkerpop.utilities
plugin activated: janusgraph.imports
plugin activated: tinkerpop.tinkergraph
gremlin> :remote connect tinkerpop.server conf/remote.yaml
==>Connected - localhost/127.0.0.1:8182
gremlin> :> graph.addVertex("name", "stephen")
==>v[256]
gremlin> :> g.V().values('name')
==>stephen
```
 `:remote` 命令告诉控制台配置一个到服务端的远程连接，该链接建立使用 `conf/remote.yaml`配置文件。 这个配置文件指向了一个运行在`localhost`的Gremlin Server 实例。 `:>`命令表示 提交命令， 它会把在那一行的Gremlin命令发送到远端服务器。 默认情况下，远程连接是无会话的，这意味着在控制台中发送的每一行都被解释为单个请求。 使用分号作为分割符，可以在一行上发送多个语句。 或者，您可以在创建连接时通过指定[session](https://tinkerpop.apache.org/docs/current/reference/#sessions)建立带有会话的控制台。 [console session](https://tinkerpop.apache.org/docs/current/reference/#console-sessions) 允许您在多行输入中重用变量。
```
gremlin> :remote connect tinkerpop.server conf/remote.yaml
==>Configured localhost/127.0.0.1:8182
gremlin> graph
==>standardjanusgraph[cql:[127.0.0.1]]
gremlin> g
==>graphtraversalsource[standardjanusgraph[cql:[127.0.0.1]], standard]
gremlin> g.V()
gremlin> user = "Chris"
==>Chris
gremlin> graph.addVertex("name", user)
No such property: user for class: Script21
Type ':help' or ':h' for help.
Display stack trace? [yN]
gremlin> :remote connect tinkerpop.server conf/remote.yaml session
==>Configured localhost/127.0.0.1:8182-[9acf239e-a3ed-4301-b33f-55c911e04052]
gremlin> g.V()
gremlin> user = "Chris"
==>Chris
gremlin> user
==>Chris
gremlin> graph.addVertex("name", user)
==>v[4344]
gremlin> g.V().values('name')
==>Chris
```
## 清理安装包

如果要重新启动并删除数据库和日志，可以用janusgraph.sh下面的clean命令。 记住一点，在运行clean操作之前，应先停止服务。
```
$ cd /Path/to/janusgraph/janusgraph-{project.version}/
$ ./bin/janusgraph.sh stop
Killing Gremlin-Server (pid 91505)...
Killing Elasticsearch (pid 91402)...
Killing Cassandra (pid 91219)...
$ ./bin/janusgraph.sh clean
Are you sure you want to delete all stored data and logs? [y/N] y
Deleted data in /Path/to/janusgraph/janusgraph-{project.version}/db
Deleted logs in /Path/to/janusgraph/janusgraph-{project.version}/log
```
## JanusGraph Server作为WebSocket终端

[入门](https://docs.janusgraph.org/master/basics/server/#getting-started) 中描述的默认配置就是一个WebSocket配置。 如果要更改默认配置以使其与自己的Cassandra或HBase环境一起使用，而不是使用快速启动环境，请按照下列步骤操作：
**配置JanusGraph Server 作为 WebSocket**

1.   首先测试本地连接JanusGraph数据库。 无论使用Gremlin Console测试连接还是从程序连接，都是可以的。 适当的更改`./conf` 目录中的属性文件中以适应你的机器环境。 例如，编辑`./conf/janusgraph-hbase.properties`并确保正确指定了storage.backend，storage.hostname和storage.hbase.table这些参数。 有关为各种存储后端配置JanusGraph的更多信息，请参见[存储后端](https://docs.janusgraph.org/master/storage-backend/)。 确保属性文件正确配置了下面几个内容：
```
gremlin.graph=org.janusgraph.core.JanusGraphFactory
```
2.  一旦本地配置完成测试，并且您有可用的属性配置文件，就将属性文件从./conf目录复制到./conf/gremlin-server目录。
```
cp conf/janusgraph-hbase.properties
conf/gremlin-server/socket-janusgraph-hbase-server.properties
```
3.  复制 ./conf/gremlin-server/gremlin-server.yaml并重命名为 socket-gremlin-server.yaml. 如果您需要参考文件的原始版本，请执行此操作。
```
cp conf/gremlin-server/gremlin-server.yaml
conf/gremlin-server/socket-gremlin-server.yaml
```
4.  修改socket-gremlin-server.yaml 文件，调整下面几项:

a. 如果您打算从非本地主机连接到JanusGraph Server，请更新主机的IP地址：
```
host: 10.10.10.100
```
b. 更新graphs部分以指向您的新属性文件，以便JanusGraph Server可以找到并连接到JanusGraph实例：
```
graphs: { graph:
    conf/gremlin-server/socket-janusgraph-hbase-server.properties}
```
5.  启动JanusGraph Server，指定您刚刚配置的yaml文件：
```
bin/gremlin-server.sh ./conf/gremlin-server/socket-gremlin-server.yaml
```
6.  JanusGraph Server现在应该以WebSocket模式运行，并且可以按照[连接到Gremlin服务器](https://docs.janusgraph.org/master/basics/server/#first-example-connecting-gremlin-server)中的说明进行测试
> 重要提示
不要使用 bin/janusgraph.sh. 这个命令会启动默认配置的服务,并且 Cassandra/Elasticsearch 环境都是独立运行的.
## JanusGraph Server 作为 HTTP 终端


[入门](https://docs.janusgraph.org/master/basics/server/#getting-started)中用的默认配置是WebSocket配置。 如果要更改默认配置以便将JanusGraph Server用作JanusGraph数据库的HTTP终端，请按照下列步骤操作：
1.  首先测试本地连接JanusGraph数据库。 无论使用Gremlin Console测试连接还是从程序连接，都是可以的。 适当的更改`./conf` 目录中的属性文件中以适应你的机器环境。 例如，编辑`./conf/janusgraph-hbase.properties`并确保正确指定了storage.backend，storage.hostname和storage.hbase.table这些参数。 有关为各种存储后端配置JanusGraph的更多信息，请参见[存储后端](https://docs.janusgraph.org/master/storage-backend/)。 确保属性文件正确配置了下面这个内容：
```
gremlin.graph=org.janusgraph.core.JanusGraphFactory
```
2.  一旦本地配置完成测试，并且您有可用的属性配置文件，就将属性文件从./conf目录复制到./conf/gremlin-server目录。
```
cp conf/janusgraph-hbase.properties conf/gremlin-server/http-janusgraph-hbase-server.properties
```
3.  复制 ./conf/gremlin-server/gremlin-server.yaml并重命名为 socket-gremlin-server.yaml. 如果您需要参考文件的原始版本，请执行此操作。
```
cp conf/gremlin-server/gremlin-server.yaml conf/gremlin-server/http-gremlin-server.yaml
```
4.  修改http-gremlin-server.yaml 文件，调整下面几项:
a. 如果您打算从非本地主机连接到JanusGraph Server，请更新主机的IP地址：
```
host: 10.10.10.100
```
b. 更新channelizer设置以指定HttpChannelizer：
```
channelizer: org.apache.tinkerpop.gremlin.server.channel.HttpChannelizer
```
c. 更新graphs部分以指向您的新属性文件，以便JanusGraph Server可以找到并连接到JanusGraph实例：
```
graphs: { graph:
    conf/gremlin-server/http-janusgraph-hbase-server.properties}
```
5.  启动JanusGraph Server, 指明你刚刚配置好的 yaml 文件:
```
bin/gremlin-server.sh ./conf/gremlin-server/http-gremlin-server.yaml
```
6. 经过上面的步骤 JanusGraph Server应该已经在HTTP模式下运行了， 可以准备开始测试了。我们可以使用 curl 命令来验证服务是否运行正常:
```
curl -XPOST -Hcontent-type:application/json -d *{"gremlin":"g.V().count()"}* [IP for JanusGraph server host](http://):8182 
```
## JanusGraph Server 双模式运行 WebSocket and HTTP Endpoint
在JanusGraph 0.2.0版本中, 你可以通过配置 gremlin-server.yaml 来让服务端使用同一个端口同时接收WebSocket 和 HTTP连接。 这可以通过更改以下任何先前示例中的channelizer来实现。
```
channelizer: org.apache.tinkerpop.gremlin.server.channel.WsAndHttpChannelizer
```
## 优化 JanusGraph Server 配置
### 通过HTTP认证
>重要提示
In the following example, credentialsDb should be different from the graph(s) you are using. It should be configured with the correct backend and a different keyspace, table, or storage directory as appropriate for the configured backend. This graph will be used for storing usernames and passwords.
在以下示例中，使用一个额外的数据库作为凭据DB。 应该为它配置合适的后端，并根据所配置的后端配置不同的键空间，表或存储目录。 这个图库将用于存储用户名和密码。

### HTTP 基础认证
要在JanusGraph Server中启用基本身份验证，请在gremlin-server.yaml中包含以下配置。
```
 authentication: {
   authenticator: org.janusgraph.graphdb.tinkerpop.gremlin.server.auth.JanusGraphSimpleAuthenticator,
   authenticationHandler: org.apache.tinkerpop.gremlin.server.handler.HttpBasicAuthenticationHandler,
   config: {
     defaultUsername: user,
     defaultPassword: password,
     credentialsDb: conf/janusgraph-credentials-server.properties
    }
 }
```
接下来验证一下基础信息认证的配置是否正确。使用下面的栗子 
```
curl -v -XPOST http://localhost:8182 -d '{"gremlin": "g.V().count()"}'
```
如果配置是正确的，此时应该返回 401 ，接着输入下面的命令
```
curl -v -XPOST http://localhost:8182 -d '{"gremlin": "g.V().count()"}' -u user:password
```
如果配置正确 ，此时应该返回 200码，并且count的结果是4 。
### 通过 WebSocket 认证
通过WebSocket进行的身份验证是通过简单身份验证和安全层机制进行的。(https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer[SASL]) 
要启用SASL身份验证，请在gremlin-server.yaml中包含以下配置
```
authentication: {
  authenticator: org.janusgraph.graphdb.tinkerpop.gremlin.server.auth.JanusGraphSimpleAuthenticator,
  authenticationHandler: org.apache.tinkerpop.gremlin.server.handler.SaslAuthenticationHandler,
  config: {
    defaultUsername: user,
    defaultPassword: password,
    credentialsDb: conf/janusgraph-credentials-server.properties
  }
}
```
>重要提示
在前面的示例中，使用一个额外的数据库作为凭据DB。 应该为它配置合适的后端，并根据所配置的后端配置不同的键空间，表或存储目录。 这个图库将用于存储用户名和密码。
如果通过gremlin控制台进行连接，则远程yaml文件应使用适当修改用户名和密码属性。
```
username: user
password: password
```
### 通过 HTTP and WebSocket认证

如果你在同时使用HTTP和WebSocket的组合channelizer，则可以使用SaslAndHMACAuthenticator来验证，它既可以通过SASL使用WebSocket验证，也可以通过基本身份验证使用HTTP验证，还能使用基于哈希的消息身份验证代码来使用HTTP验证 (https://en.wikipedia.org/wiki/Hash-based_message_authentication_code[HMAC])。 HMAC旨在通过HTTP使用基于令牌的身份验证。 您首先通过/session端点获取令牌，然后使用该令牌进行身份验证。 它用于分摊使用基本身份验证加密密码所花费的时间。
要使该方式生效，gremlin-server.yaml应该包含下面这些配置项：
```
authentication: {
  authenticator: org.janusgraph.graphdb.tinkerpop.gremlin.server.auth.SaslAndHMACAuthenticator,
  authenticationHandler: org.janusgraph.graphdb.tinkerpop.gremlin.server.handler.SaslAndHMACAuthenticationHandler,
  config: {
    defaultUsername: user,
    defaultPassword: password,
    hmacSecret: secret,
    credentialsDb: conf/janusgraph-credentials-server.properties
  }
}
```
>重要提示
在前面的示例中，使用一个额外的数据库作为凭据DB。 应该为它配置合适的后端，并根据所配置的后端配置不同的键空间，表或存储目录。 这个图库将用于存储用户名和密码。

>重要提示
在这里注意hmacSecret。 如果您希望能够在每台服务器上使用相同的HMAC令牌，则在所有运行的JanusGraph服务器上hmacSecret应该是相同的。
在HTTP场景下使用HMAC身份验证，会创建一个/session终端，该终端提供一个验证令牌，该令牌默认情况下在一小时后过期。 令牌的超时时长可以通过authentication.config映射中的tokenTimeout配置选项进行配置。 该值是一个Long值，以毫秒为单位。

您可以使用curl向/session终端发出get请求来获取令牌。 方法如下：
```
curl http://localhost:8182/session -XGET -u user:password

{"token": "dXNlcjoxNTA5NTQ2NjI0NDUzOkhrclhYaGhRVG9KTnVSRXJ5U2VpdndhalJRcVBtWEpSMzh5WldqRTM4MW89"}
```

然后，可以通过使用“ Authorization: Token”标头将该令牌用于身份验证。 例如
```
curl -v http://localhost:8182/session -XPOST -d '{"gremlin": "g.V().count()"}' -H "Authorization: Token dXNlcjoxNTA5NTQ2NjI0NDUzOkhrclhYaGhRVG9KTnVSRXJ5U2VpdndhalJRcVBtWEpSMzh5WldqRTM4MW89"
```
### TinkerPop Gremlin服务与JanusGraph一起使用

由于JanusGraph Server是集成了TinkerPop Gremlin服务，其中通过配置文件来进行配置，因此可以单独下载与版本兼容的TinkerPop Gremlin服务，并将其与JanusGraph一起使用。 要想开始使用，需要下载适当版本的Gremlin Server，该版本需要与所使用的JanusGraph版本匹配(3.4.5)。
>重要提示
除非特别说明，否则本节中对文件路径的任何引用均指Gremlin Server TinkerPop下的路径，而不是JanusGraph Server的JanusGraph下的路径。

配置独立的Gremlin服务以与JanusGraph一起使用跟配置打包的JanusGraph服差不多。 您应该熟悉[图形配置](https://tinkerpop.apache.org/docs/3.4.5/reference/#_configuring_2)。 基本上，Gremlin Server的 yaml文件指向特定于图的配置文件，该文件用于实例化将要托管的JanusGraph实例。 为了实例化这些Graph，Gremlin Server要求JanusGraph的相应库和依赖项在其类路径上可用。

为了方便演示，这些说明将概述如何在Gremlin Server中为JanusGraph配置BerkeleyDB后端。 如前所述，Gremlin Server在其类路径上需要JanusGraph依赖项。 调用以下命令以将`$VERSION`替换为要使用的JanusGraph版本：
```
bin/gremlin-server.sh -i org.janusgraph janusgraph-all $VERSION
```

此过程完成后，Gremlin Server现在应该可以使用所有JanusGraph依赖项，因此可以实例化`JanusGraph` 对象。
>重要提示
The above command uses Groovy Grape and if it is not configured properly 
上面的命令使用了Groovy Grape，如果未正确配置，则可能会导致下载错误。 请参阅TinkerPop文档的[本节](https://tinkerpop.apache.org/docs/3.4.5%20/reference/#gremlin-applications) 以获取有关设置~/.groovy/grapeConfig.xml的更多信息。 

创建一个名为`GREMLIN_SERVER_HOME/conf/janusgraph.properties` 的文件，其内容如下：
```
gremlin.graph=org.janusgraph.core.JanusGraphFactory
storage.backend=berkeleyje
storage.directory=db/berkeley
```

其他后端的配置也与此类似。 请参阅[存储后端](https://docs.janusgraph.org/master/storage-backend/)。 如果使用Cassandra，请在 `janusgraph.properties` 文件中使用Cassandra配置选项。 唯一保持不变的重要部分是 `gremlin.graph`设置，该设置应始终使用 `JanusGraphFactory`。 此设置告诉Gremlin Server如何实例化JanusGraph实例。
下一步创建一个文件，名为 `GREMLIN_SERVER_HOME/conf/gremlin-server-janusgraph.yaml` ，包含如下内容:
```
host: localhost
port: 8182
graphs: {
  graph: conf/janusgraph.properties}
scriptEngines: {
  gremlin-groovy: {
    plugins: { org.janusgraph.graphdb.tinkerpop.plugin.JanusGraphGremlinPlugin: {},
               org.apache.tinkerpop.gremlin.server.jsr223.GremlinServerGremlinPlugin: {},
               org.apache.tinkerpop.gremlin.tinkergraph.jsr223.TinkerGraphGremlinPlugin: {},
               org.apache.tinkerpop.gremlin.jsr223.ImportGremlinPlugin: {classImports: [java.lang.Math], methodImports: [java.lang.Math#*]},
               org.apache.tinkerpop.gremlin.jsr223.ScriptFileGremlinPlugin: {files: [scripts/empty-sample.groovy]}}}}
serializers:
  - { className: org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0, config: { ioRegistries: [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistry] }}
  - { className: org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0, config: { serializeResultToString: true }}
  - { className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV3d0, config: { ioRegistries: [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistry] }}
metrics: {
  slf4jReporter: {enabled: true, interval: 180000}}
```
#### 此配置文件有几个重要部分与JanusGraph相关。

1. 在图映射中，有一个名为graph的键，其值为conf/janusgraph.properties。 它告诉Gremlin Server实例化一个名为“graph”的Graph实例，并使用conf/janusgraph.properties文件对其进行配置。 “ graph”键成为Gremlin Server中Graph实例的唯一名称，可以在提交给它的脚本中引用它。

2. 在插件列表中，有对JanusGraphGremlinPlugin的引用，它告诉Gremlin Server初始化“ JanusGraph插件”。 “ JanusGraph插件”将自动导入JanusGraph特定的类以供在脚本中使用。
3. 注意脚本键和对scripts/janusgraph.groovy的引用。 该Groovy文件是Gremlin Server和该特定ScriptEngine的初始化脚本。 使用以下内容创建脚本/janusgraph.groovy：
```
def globals = [:]
globals << [g : graph.traversal()]
```
上面的脚本创建了一个名为globals的Map，并为其分配了一个键/值对。 键是g，其值是从图中生成的TraversalSource，该图是在其配置文件中为Gremlin Server配置的。 此时，提供给Gremlin Server的脚本可以使用两个全局变量graph和g。

至此，Gremlin Server已配置好，可用于连接到新的或现有的JanusGraph数据库。 启动服务：
```
$ bin/gremlin-server.sh conf/gremlin-server-janusgraph.yaml
[INFO] GremlinServer -
         \,,,/
         (o o)
-----oOOo-(3)-oOOo-----

[INFO] GremlinServer - Configuring Gremlin Server from conf/gremlin-server-janusgraph.yaml
[INFO] MetricManager - Configured Metrics Slf4jReporter configured with interval=180000ms and loggerName=org.apache.tinkerpop.gremlin.server.Settings$Slf4jReporterMetrics
[INFO] GraphDatabaseConfiguration - Set default timestamp provider MICRO
[INFO] GraphDatabaseConfiguration - Generated unique-instance-id=7f0000016240-ubuntu1
[INFO] Backend - Initiated backend operations thread pool of size 8
[INFO] KCVSLog$MessagePuller - Loaded unidentified ReadMarker start time 2015-10-02T12:28:24.411Z into org.janusgraph.diskstorage.log.kcvs.KCVSLog$MessagePuller@35399441
[INFO] GraphManager - Graph [graph] was successfully configured via [conf/janusgraph.properties].
[INFO] ServerGremlinExecutor - Initialized Gremlin thread pool.  Threads in pool named with pattern gremlin-*
[INFO] ScriptEngines - Loaded gremlin-groovy ScriptEngine
[INFO] GremlinExecutor - Initialized gremlin-groovy ScriptEngine with scripts/janusgraph.groovy
[INFO] ServerGremlinExecutor - Initialized GremlinExecutor and configured ScriptEngines.
[INFO] ServerGremlinExecutor - A GraphTraversalSource is now bound to [g] with graphtraversalsource[standardjanusgraph[berkeleyje:db/berkeley], standard]
[INFO] AbstractChannelizer - Configured application/vnd.gremlin-v3.0+gryo with org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0
[INFO] AbstractChannelizer - Configured application/vnd.gremlin-v3.0+gryo-stringd with org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0
[INFO] GremlinServer$1 - Gremlin Server configured with worker thread pool of 1, gremlin pool of 8 and boss thread pool of 1.
[INFO] GremlinServer$1 - Channel started at port 8182.
```
以下部分说明了如何连接到正在运行的服务上。

#### 通过Gremlin服务连接到JanusGraph

Gremlin Server启动后将准备好侦听WebSocket连接。 测试连接的最简单方法是使用Gremlin Console。

根据这里的步骤 [Connecting to Gremlin Server](https://docs.janusgraph.org/master/connecting/) 来检查Gremlin Server 是否工作正常。

>重要提示
您应该理解的区别是，在使用JanusGraph Server时，Gremlin Console从JanusGraph下启动，而当按照此处针对独立Gremlin Server的测试说明进行操作时，Gremlin Console从TinkerPop下启动。
```
GryoMapper mapper = GryoMapper.build().addRegistry(JanusGraphIoRegistry.INSTANCE).create();
Cluster cluster = Cluster.build().serializer(new GryoMessageSerializerV3d0(mapper)).create();
Client client = cluster.connect();
client.submit("g.V()").all().get();
```
通过将JanusGraphIoRegistry添加到org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0中，驱动程序将知道如何正确地反序列化JanusGraph返回的自定义数据类型。
## 扩展JanusGraph服务
通过实现Gremlin Server提供的接口，可以通过其他通信方式扩展Gremlin Server，并进行升级。想了解这个，可以在对应的TinkerPop文档中查看更多详细信息。