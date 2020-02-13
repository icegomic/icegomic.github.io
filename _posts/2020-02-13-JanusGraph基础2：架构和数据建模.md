---
layout:     post
title:      JanusGraph基础2：架构和数据建模
subtitle:   
date:       2020-02-13
author:     IceGomic
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - 知识图谱
    - 图数据库
    - NLP
---


> 本文首次发布于 [IceGomic Blog](http://icegomic.github.io), 作者 [@IceGomic](http://github.com/icegomic) ,转载请保留原文链接.

每个JanusGraph图都有一个架构，该架构包括其中使用的边标签，属性键和顶点标签。 可以显式或隐式定义JanusGraph架构。 当然，鼓励用户在应用程序开发期间明确定义图模式。 显式定义架构可以增强图应用程序的鲁棒性，可以大大改善软件开发的合作效率。 注意，JanusGraph架构可以随着时间的推移而演变，而不会中断正常的数据库操作。 扩展架构不会降低查询的速度，也不需要数据库停机。

在创建一个图的时候，架构类型（边缘标签，属性键或顶点标签）即会分配给图形中的元素-分别为边缘，属性或顶点。 不能为特定元素更改分配的架构类型。 这确保了稳定类型系统对问题追根溯源。

除了本节中介绍的架构定义选项外，架构类型还提供了性能调整选项，这些选项在[高级架构](https://docs.janusgraph.org/master/advanced-topics/advschema/)中进行了讨论。
## 展示架构信息

有一些方法可以使用管理API查看架构中的特定元素。 这些方法是`mgmt.printIndexes()`，`mgmt.printPropertyKeys()`，`mgmt.printVertexLabels()`和`mgmt.printEdgeLabels()`。 还有一种方法可以显示所有这些内容，这个方法名为`printSchema()`。
```
mgmt = graph.openManagement()
mgmt.printSchema()
```
## 定义边标签

连接两个顶点的边都有一个标签，用于定义关系的含义。 例如，在顶点A和顶点B之间的边被标记为`朋友`，这表明A和B两个人之间是朋友关系。

要定义边的标签，请在打开的图形或管理事务上调用`makeEdgeLabel(String)`，其中，参数为边标签的名称。 边标签名称在图中必须是唯一。 此方法返回一个边标签生成器，该方法允许定义其多重性。 边标签的多重性定义了该标签在所有边上的多重性约束，即，成对的顶点之间的最大边数。 JanusGraph支持以下多重性设置。

### 边缘标签多样性
MULTI: 允许在任意一对顶点之间使用带有同一标签的多条边。换句话说，该图是关于这种标签的多重图。边缘多重性没有任何限制。
SIMPLE: 在任何一对顶点之间至多允许有该标签的一条边。换句话说，该图是关于此标签的简单图。确保给定标签和成对的顶点的边是唯一的。
MANY2ONE: 在图形的任意顶点上至多允许一个带有该标签的输出边，但对输入边缘不施加任何限制。边标签`母亲`是MANY2ONE多样性的一个例子，因为每个人最多只有一个母亲，但是母亲可以有多个孩子。
ONE2MANY: 在图形的任意顶点上至多允许一个带有该标签的输入边，但对输出边没有任何限制。边缘标签`winnerOf`是一个具有ONE2MANY多重性的示例，因为每个竞赛最多只能有一个人赢，但一个人可以赢得多个竞赛。
ONE2ONE: 在图形的任意顶点上至多允许一个带有该标签的输入边和一个输出边。由于一个人只能与另外的“一个人”结婚，标签“ marriedTo”是ONE2ONE多重性的一个示例。

默认多重性设置是MULTI。边标签的定义是通过在构建器上调用`make()`方法完成的，该方法返回定义的边标签，如以下示例所示。
```
mgmt = graph.openManagement()
follow = mgmt.makeEdgeLabel('follow').multiplicity(MULTI).make()
mother = mgmt.makeEdgeLabel('mother').multiplicity(MANY2ONE).make()
mgmt.commit()
```
## 定义属性键
顶点和边上的属性是键值对。 例如，属性名称="Daniel"具有键和值“Daniel”。 属性键是JanusGraph架构的一部分，可以限制允许的数据类型和值的基数。

要定义属性键，请在打开的图形或管理事务上调用makePropertyKey(String)并提供属性键的名称作为参数。 属性键名称在图中必须是唯一的，建议避免在属性名称中使用空格或特殊字符。 此方法返回属性键的生成器。
```
- Note

在创建属性键时，可以考虑同时创建图形索引以获得更好的性能，请参见[索引性能](https://docs.janusgraph.org/master/index-management/index-performance/).

```
### 属性键数据类型

使用dataType(Class)定义属性键的数据类型。 JanusGraph将强制与键关联的所有值都具有配置的相同的数据类型，从而确保添加到图的数据有效。 例如，可以定义`name`键包含一个String数据类型。

把数据类型定义为`Object.class`以允许任何（可序列化）类型值与键相关联。 但是，我们还是建议尽可能使用具体的数据类型。 配置的数据类型必须是具体的类，而不是接口或抽象类。 JanusGraph强制执行类相等性，因此不允许添加已配置数据类型的子类。

JanusGraph支持以下数据类型。

原生的JanusGraph数据类型

| 名称 | 描述 |
| --- | --- |
| String | 字符串 |
| Character | 单个字符 |
| Boolean | true or false |
| Byte | byte value |
| Short | short value |
| Integer | integer value |
| Long | long value |
| Float | 4 byte floating point number |
| Double | 8 byte floating point number |
| Date | Specific instant in time (`java.util.Date`) |
| Geoshape | 地理形状，如点，圈，框|
| UUID | 通用唯一标识符 (`java.util.UUID`) |

### 属性键基数

使用基数（Cardinality）定义与任何给定顶点上的键关联的值的允许基数。
*   **SINGLE**: 此键的每个元素最多有一个值。 换句话说，图中的所有元素的键→值映射都是唯一的。 属性键`birthDate`是SINGLE基数的示例，因为每个人的出生日期都是唯一的。
*   **LIST**: 允许该键的每个元素具有任意数量的值。 换句话说，该键关联了允许重复值的值列表。 假设我们将传感器建模为图形中的顶点，则属性键`sensorReading`就是一个具有LIST基数的示例，可以记录很多（可能重复的）传感器读数。
*   **SET**: 此类键的每个元素允许多个值，但不允许重复值。 换句话说，键与一组不重复的值相关联。 如果我们要捕获个人的所有姓名（包括昵称，小名等），则属性键的``姓名''具有SET基数。

默认基数设置为SINGLE。 请注意，在边和属性上使用的属性键的基数为SINGLE。 不支持在边或属性上为单个键附加多个值。
```
mgmt = graph.openManagement()
birthDate = mgmt.makePropertyKey('birthDate').dataType(Long.class).cardinality(Cardinality.SINGLE).make()
name = mgmt.makePropertyKey('name').dataType(String.class).cardinality(Cardinality.SET).make()
sensorReading = mgmt.makePropertyKey('sensorReading').dataType(Double.class).cardinality(Cardinality.LIST).make()
mgmt.commit()
```
## 关系类型

边标签和属性键合起来共同称为关系类型。 关系类型的名称在图中必须是唯一的，这意味着属性键和边标签不能具有相同的名称。 JanusGraph API中提供了一些方法来查询是否存在或检索关系类型，这些类型同时包含属性键和边标签。
```
mgmt = graph.openManagement()
if (mgmt.containsRelationType('name'))
    name = mgmt.getPropertyKey('name')
mgmt.getRelationTypes(EdgeLabel.class)
mgmt.commit()
```
## 定义顶点标签

像边缘一样，顶点也有标签。 与边缘标签不同，顶点标签是可选的。 顶点标签可用于区分不同类型的顶点，例如 用户顶点和产品顶点。

尽管标签在概念和数据模型级别是可选的，但在内部实现的时候，JanusGraph实际上会为所有顶点分配标签。 由addVertex方法创建的顶点使用JanusGraph的默认标签。

要创建标签，请在打开的图形或管理事务上调用makeVertexLabel(String).make()并提供顶点标签的名称作为参数。 顶点标签名称在图中必须唯一。

```
mgmt = graph.openManagement()
person = mgmt.makeVertexLabel('person').make()
mgmt.commit()
// Create a labeled vertex
person = graph.addVertex(label, 'person')
// Create an unlabeled vertex
v = graph.addVertex()
graph.tx().commit()
```
## 自动架构生成器

如果边标签，属性键或顶点标签未明确定义，那么在添加边，顶点或设置属性的时候期间，将为它们添加隐式定义。 的DefaultSchemaMaker为JanusGraph图配置定义了这些类型。

默认情况下，隐式创建的这些类型，边缘标签多重性会设置为MULTI，而属性键基数设置为SINGLE和数据类型默认Object.class。 用户可以通过实现和注册自己的DefaultSchemaMaker来控制自动架构元素的创建。

在为不同于SINGLE的顶点属性定义基数时，应将基数用于第一个查询（即定义新顶点属性键的查询）中所有顶点属性值。

强烈建议通过在JanusGraph图形配置中设置schema.default = none来显式定义所有架构元素并禁用自动架构创建。
#### 修改架构元素

边标签，属性键或顶点标签的定义一旦提交到图形中就无法更改。 但是，可以通过JanusGraphManagement.changeName(JanusGraphSchemaElement, String) 更改架构元素的名称，如以下所示，将属性键位置重命名为location。
```
mgmt = graph.openManagement()
place = mgmt.getPropertyKey('place')
mgmt.changeName(place, 'location')
mgmt.commit()
```

请注意，架构名称更改在群集中当前正在运行的事务和其他JanusGraph图实例中可能不会立即可见。 尽管通过存储后端向所有JanusGraph实例宣布了架构名称更改，但是架构更改可能需要一段时间才能生效，并且如果某些故障情况（例如网络问题，如果它们与重命名冲突）出现，则可能需要重新启动实例。 因此，用户必须确保以下任一条件成立：

重命名的标签或键不会立即处于可用状态（即已写入或读取），必须直到所有JanusGraph实例都知道名称更改后才会使用。
正在运行的事务会主动适应短暂的中间时间段，根据特定的JanusGraph实例和名称更改公告的状态，旧名称或新名称均有效。 例如，这可能意味着事务可以同时查询两个名称。

如果需要重新定义现有的架构类型，建议将这种类型的名称更改为当前没有在使用的名称（最好永远不会使用的名称）。 之后，可以用原来的名称来定义新的标签或键，从而有效地替换旧的标签或键。 但是，请注意，这不会影响以前已经写好的顶点，边或属性使用现有类型。 重新定义现有图元素不支持在线处理，必须通过批处理图转换来完成。
### 架构约束

架构的定义允许用户配置显式属性和连接约束。 可以将属性绑定到特定的顶点标签和/或边标签上。 此外，连接约束允许用户显式定义可以通过边标签连接的两个顶点标签。 这些约束可用于确保图匹配给定的域模型。 例如，对于“众神”的图，一个神可以是另一个神的兄弟，但不能是怪物的兄弟，一个神可以有财产年龄，但是位置不能有财产年龄。 这些约束默认情况下处于禁用状态。

通过设置schema.constraints=true来启用这些架构约束。 此设置取决于设置schema.default。 如果配置schema.default设置为none，则将因架构约束冲突而抛出IllegalArgumentException。 如果未将schema.default设置为none，则将自动创建架构约束，但不会引发任何异常。 激活架构约束对现有数据没有影响，因为这些架构约束仅在插入过程中应用。 因此，这些约束完全不会影响数据的读取。

可以使用JanusGraphManagement.addProperties(VertexLabel, PropertyKey...)将多个属性绑定到一个顶点上，例如：
```
mgmt = graph.openManagement()
person = mgmt.makeVertexLabel('person').make()
name = mgmt.makePropertyKey('name').dataType(String.class).cardinality(Cardinality.SET).make()
birthDate = mgmt.makePropertyKey('birthDate').dataType(Long.class).cardinality(Cardinality.SINGLE).make()
mgmt.addProperties(person, name, birthDate)
mgmt.commit()
```

可以使用JanusGraphManagement.addProperties(EdgeLabel, PropertyKey...)将多个属性绑定到一条边上，例如：
```
mgmt = graph.openManagement()
follow = mgmt.makeEdgeLabel('follow').multiplicity(MULTI).make()
name = mgmt.makePropertyKey('name').dataType(String.class).cardinality(Cardinality.SET).make()
mgmt.addProperties(follow, name)
mgmt.commit()
```

可以使用JanusGraphManagement.addConnection(EdgeLabel, VertexLabel out, VertexLabel in)方法在传出点，传入点和边缘之间定义连接，例如：
```
mgmt = graph.openManagement()
person = mgmt.makeVertexLabel('person').make()
company = mgmt.makeVertexLabel('company').make()
works = mgmt.makeEdgeLabel('works').multiplicity(MULTI).make()
mgmt.addConnection(works, person, company)
mgmt.commit()
```