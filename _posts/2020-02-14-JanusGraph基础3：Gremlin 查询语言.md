---
layout:     post
title:      JanusGraph基础3：Gremlin 查询语言
subtitle:   
date:       2020-02-14
author:     IceGomic
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - 知识图谱
    - 图数据库
    - NLP
---


> 本文首次发布于 [IceGomic Blog](http://icegomic.github.io), 作者 [@IceGomic](http://github.com/icegomic) ,转载请保留原文链接.

![Gremlin](https://upload-images.jianshu.io/upload_images/100763-96d5cbe3879f3b18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[Gremlin](https://tinkerpop.apache.org/gremlin.html)是JanusGraph的查询语言，用于从图中检索数据和修改图中的数据。 Gremlin是一种面向路径的语言，可以简洁地表示复杂的图遍历和变种操作。 Gremlin是一种[功能语言](https://en.wikipedia.org/wiki/Functional_programming)，遍历运算符被链接在一起以形成类似路径的表达式。 例如，“从大力神Hercules出发，先走到他的父亲，再走到他父亲的父亲，然后返回祖父的名字。”

Gremlin是[Apache TinkerPop](https://tinkerpop.apache.org/)的组件。 它独立于JanusGraph开发，大多数图形数据库都支持该查询语言。 通过使用Gremlin查询语言在JanusGraph之上构建应用程序，用户可以避免供应商锁定，因为他们的应用程序可以迁移到支持Gremlin的其他图形数据库。

本节是Gremlin查询语言的简要概述。 有关Gremlin的更多信息，请参考以下资源：

*   [Practical Gremlin](https://kelvinlawrence.net/book/Gremlin-Graph-Guide.html):Kelvin R. Lawrence提供的的在线书籍，深入讲述了Gremlin与JanusGraph的交互。
*   [Complete Gremlin Manual](https://tinkerpop.apache.org/docs/3.4.5/reference/): 所有Gremlin步骤的参考手册。

*   [Gremlin Console Tutorial](https://tinkerpop.apache.org/docs/3.4.5/tutorials/the-gremlin-console/): 了解如何有效使用Gremlin Console交互地遍历和分析图形。

*   [Gremlin Recipes](https://tinkerpop.apache.org/docs/3.4.5/recipes/): Gremlin的最佳做法和常见遍历模式的集合。

*   [Gremlin Language Drivers](https://tinkerpop.apache.org/index.html#language-drivers): 使用不同的编程语言连接到Gremlin Server，包括Go，JavaScript，.NET/C#，PHP，Python，Ruby，Scala和TypeScript。

*   [Gremlin Language Variants](https://tinkerpop.apache.org/docs/3.4.5/tutorials/gremlin-language-variants/): 了解如何将Gremlin嵌入宿主编程语言。

*   [Gremlin for SQL developers](http://sql2gremlin.com/): Learn Gremlin using typical patterns found when querying data with SQL.
学习使用SQL查询数据时，Gremlin使用的典型模式。
除了上面的资源，这里还有一个, [Connecting to JanusGraph](https://docs.janusgraph.org/master/connecting/) 解释了如何在不同的编程语言中使用Gremlin来查询JanusGraph Server。
## 遍历方法的介绍

Gremlin查询语句是从左到右来解析的的一串操作/函数。 以下是在[入门](https://docs.janusgraph.org/master/#getting-started)中讨论的*众神之图*数据集下面提供的一个简单的祖父查询示例。
```
gremlin> g.V().has('name', 'hercules').out('father').out('father').values('name')
==>saturn
```
让我们来解释一下上面的语句：

g: 我们当前在使用的图
V: 该图中的全部顶点
has('name', 'hercules'): 过滤出名字属性为"hercules" 的顶点（有且只有一个）
out('father'): 遍历从顶点Hercules出发的父亲边。
‘out('father')`: 遍历从顶点Hercules的父亲出发的父亲边。 (i.e. Jupiter).
name: 获取 "hercules" 顶点的祖父的名称属性。

这些步骤加在一起构成了类似路径的遍历查询。 每个步骤都可以分解，并且可以证明其结果。 这种构造遍历/查询的样式在构造较大，较复杂的查询时非常有用。
```
gremlin> g
==>graphtraversalsource[janusgraph[cql:127.0.0.1], standard]
gremlin> g.V().has('name', 'hercules')
==>v[24]
gremlin> g.V().has('name', 'hercules').out('father')
==>v[16]
gremlin> g.V().has('name', 'hercules').out('father').out('father')
==>v[20]
gremlin> g.V().has('name', 'hercules').out('father').out('father').values('name')
==>saturn
```
For a sanity check, it is usually good to look at the properties of each return, not the assigned long id.
对于健全性检查，通常最好查看每个返回的属性，而不是分配的long id。
```
gremlin> g.V().has('name', 'hercules').values('name')
==>hercules
gremlin> g.V().has('name', 'hercules').out('father').values('name')
==>jupiter
gremlin> g.V().has('name', 'hercules').out('father').out('father').values('name')
==>saturn
```

注意相关遍历，该遍历显示了大力神Hercules的整个父亲家谱分支。 提供这种更复杂的遍历是为了演示语言的灵活性和表达性。 对Gremlin的熟练掌握为JanusGraph用户提供了流畅浏览基础图形结构的能力。
```
gremlin> g.V().has('name', 'hercules').repeat(out('father')).emit().values('name')
==>jupiter
==>saturn
```
下面提供了一些遍历示例。
```
gremlin> hercules = g.V().has('name', 'hercules').next()
==>v[1536]
gremlin> g.V(hercules).out('father', 'mother').label()
==>god
==>human
gremlin> g.V(hercules).out('battled').label()
==>monster
==>monster
==>monster
gremlin> g.V(hercules).out('battled').valueMap()
==>{name=nemean}
==>{name=hydra}
==>{name=cerberus}
```

每个步骤（用分隔号表示）都是对从上一步发出的对象进行操作的方法。 Gremlin语言有许多步骤（请参阅[Gremlin步骤](https://tinkerpop.apache.org/docs/3.4.5/reference/#graph-traversal-steps
)）。 通过简单地更改一个步骤或步骤的顺序，可以实现不同的遍历语义。 以下示例返回了与大力神干过的相同怪物的所有人的姓名，大力神除外（即“同战者”或“盟友”）。

假设*众神之图*只有一个战士（大力神Hercules），下面的栗子则将另一个战斗者（为方便起见）添加到图中，Gremlin展示了如何将顶点和边添加到图中。
```
gremlin> theseus = graph.addVertex('human')
==>v[3328]
gremlin> theseus.property('name', 'theseus')
==>null
gremlin> cerberus = g.V().has('name', 'cerberus').next()
==>v[2816]
gremlin> battle = theseus.addEdge('battled', cerberus, 'time', 22)
==>e[7eo-2kg-iz9-268][3328-battled->2816]
gremlin> battle.values('time')
==>22
```

添加顶点时，顶点标签是可选的。 添加边时必须指定边标签。 可以在顶点和边上设置作为键值对的属性。 当使用SET或LIST基数定义属性键时，在将相应的属性添加到顶点时必须使用addProperty。
```
gremlin> g.V(hercules).as('h').out('battled').in('battled').where(neq('h')).values('name')
==>theseus
```

上面的示例写了4个链式方法：out，in，except和values（name是values('name')的简写）。 每个函数的功能在下面逐项列出，其中V是顶点，U是对象，其中V是U的子集。
1. out: V -> V
2. in: V -> V
3. except: U -> U
4. values: V -> U

将函数链接在一起时，后一个的传入类型必须匹配前一个的传出类型，其中U匹配任何内容。 因此，上面的“共同战斗/同盟”遍历是正确的。

注意
本节主要介绍了在Gremlin控制台中使用的Gremlin-Groovy语言实现操作。 请参阅[Connecting to JanusGraph](https://docs.janusgraph.org/master/connecting/)以获取有关使用Groovy之外的其他语言独立于Gremlin Console连接到JanusGraph的信息。
## 迭代遍历

Gremlin控制台的一项便利功能是，它会自动迭代从gremlin>提示符下执行的查询的所有结果。 这在[REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) 环境中效果很好，因为它以字符串形式显示结果。 当您过渡到编写Gremlin应用程序时，了解如何显式遍历非常重要，因为应用程序的遍历不会自动进行迭代。 这些是迭代[Traversal](https://tinkerpop.apache.org/javadocs/3.4.5/full/org/apache/tinkerpop/gremlin/process/traversal/Traversal.html)的一些常用方法：
*   `iterate()` - 预期结果为零或可以忽略。
*   `next()` - 过去一个结果。 首先检查是否有下一个 `hasNext()`.
*   `next(int n)` - 获取下面 `n` 个结果. 先检查是否有下一个 `hasNext()`.
*   `toList()` - 获取所有结果并转换成list。如果没有结果，将返回一个空list。

下面显示了一段Java代码示例来演示这些概念：
```
Traversal t = g.V().has("name", "pluto"); // 定义一个遍历操作
// 此时操作尚未执行
Vertex pluto = null;
if (t.hasNext()) { // 检查是否有下一个结果
    pluto = g.V().has("name", "pluto").next(); // 获取一个结果
    g.V(pluto).drop().iterate(); // Execute a traversal to drop pluto from graph
}
// Note the traversal can be cloned for reuse
Traversal tt = t.asAdmin().clone();
if (tt.hasNext()) {
    System.err.println("pluto was not dropped!");
}
List<Vertex> gods = g.V().hasLabel("god").toList(); // Find all the gods
```