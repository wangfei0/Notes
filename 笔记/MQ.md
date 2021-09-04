# RabbitMQ（了解即可）

## 1. 概念 

- 异步
- 消峰
- 解耦

RabbitMQ 是采用 Erlang 语言实现 AMQP(Advanced Message Queuing Protocol，高级消息队列协议）的消息中间件，它最初起源于金融系统，用于在分布式系统中存储转发消息。

**特点：**

- 可靠性：
- 灵活的路由：
- 扩展性：多个RabbitMQ节点可以组成一个集群
- 高可用性：
- 支持多种协议
- 多语言客户端：
- 易用的管理界面：
- 插件机制：

**核心概念**

RabbitMQ 整体上是一个生产者与消费者模型，主要负责接收、存储和转发消息。

![图1-RabbitMQ 的整体模型架构](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/96388546.jpg)

- **Producer(生产者)** :生产消息的一方（邮件投递者）
- **Consumer(消费者)** :消费消息的一方（邮件收件人）

消息一般由 2 部分组成：**消息头**（或者说是标签 Label）和 **消息体**。消息体也可以称为 payLoad ,消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括 routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。生产者把消息交由 RabbitMQ 后，RabbitMQ 会根据消息头把消息发送给感兴趣的 Consumer(消费者)。

- Exchange(交换器)

在 RabbitMQ 中，消息并不是直接被投递到 **Queue(消息队列)** 中的，中间还必须经过 **Exchange(交换器)** 这一层，**Exchange(交换器)** 会把我们的消息分配到对应的 **Queue(消息队列)** 中。

**Exchange(交换器)** 用来接收生产者发送的消息并将这些消息路由给服务器中的队列中，如果路由不到，或许会返回给 **Producer(生产者)** ，或许会被直接丢弃掉 。这里可以将RabbitMQ中的交换器看作一个简单的实体。

**RabbitMQ 的 Exchange(交换器) 有4种类型，不同的类型对应着不同的路由策略**：**direct(默认)\**，**fanout**, **topic**, 和 **headers**，不同类型的Exchange转发消息的策略有所区别。

- Queue(消息队列)

**Queue(消息队列)** 用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

**多个消费者可以订阅同一个队列**，这时队列中的消息会被平均分摊（Round-Robin，即轮询）给多个消费者进行处理，而不是每个消费者都收到所有的消息并处理，这样避免的消息被重复消费。

- #### Broker（消息中间件的服务节点）

对于 RabbitMQ 来说，一个 RabbitMQ Broker 可以简单地看作一个 RabbitMQ 服务节点，或者RabbitMQ服务实例。大多数情况下也可以将一个 RabbitMQ Broker 看作一台 RabbitMQ 服务器。

![消息队列的运转过程](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/67952922.jpg)

### 1.1 交换器类型Exchange Types

RabbitMQ 常用的 Exchange Type 有 **fanout**、**direct**、**topic**、**headers** 这四种。

##### [① fanout](https://snailclimb.gitee.io/javaguide/#/docs/system-design/data-communication/rabbitmq?id=①-fanout)

fanout 类型的Exchange路由规则非常简单，它会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中，不需要做任何判断操作，所以 fanout 类型是所有的交换机类型里面速度最快的。fanout 类型常用来广播消息。

##### [② direct](https://snailclimb.gitee.io/javaguide/#/docs/system-design/data-communication/rabbitmq?id=②-direct)

direct 类型的Exchange路由规则也很简单，它会把消息路由到那些 Bindingkey （绑定key）与 RoutingKey （路由key）完全匹配的 Queue 中。

![direct 类型交换器](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/37008021.jpg)

direct 类型常用在处理有优先级的任务，根据任务的优先级把消息发送到对应的队列，这样可以指派更多的资源去处理高优先级的队列。

##### [③ topic](https://snailclimb.gitee.io/javaguide/#/docs/system-design/data-communication/rabbitmq?id=③-topic)

正则匹配。

- RoutingKey 为一个点号“．”分隔的字符串（被点号“．”分隔开的每一段独立的字符串称为一个单词），如 “com.rabbitmq.client”、“java.util.concurrent”、“com.hidden.client”;
- BindingKey 和 RoutingKey 一样也是点号“．”分隔的字符串；
- BindingKey 中可以存在两种特殊字符串“*”和“#”，用于做模糊匹配，其中“*”用于匹配一个单词，“#”用于匹配多个单词(可以是零个)。

![topic 类型交换器](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/73843.jpg)

##### [④ headers(不推荐)](https://snailclimb.gitee.io/javaguide/#/docs/system-design/data-communication/rabbitmq?id=④-headers不推荐)

headers 类型的交换器不依赖于路由键的匹配规则来路由消息，而是根据发送的消息内容中的 headers 属性进行匹配。

headers 类型的交换器性能会很差，而且也不实用，基本上不会看到它的存在。

#  RocketMQ（重要）

- 异步
- 消峰
- 解耦

## 概念

`RocketMQ` 是一个 **队列模型** 的消息中间件，具有**高性能、高可靠、高实时、分布式** 的特点。它是一个采用 `Java` 语言开发的分布式的消息系统，由阿里巴巴团队开发，在2016年底贡献给 `Apache`，成为了 `Apache` 的一个顶级项目。 在阿里内部，`RocketMQ` 很好地服务了集团大大小小上千个应用，在每年的双十一当天，更有不可思议的万亿级消息通过 `RocketMQ` 流转。

![img](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/16ef383d3e8c9788.jpg)

- `Producer Group` 生产者组： 代表某一类的生产者，比如我们有多个秒杀系统作为生产者，这多个合在一起就是一个 `Producer Group` 生产者组，它们一般生产相同的消息。
- `Consumer Group` 消费者组： 代表某一类的消费者，比如我们有多个短信系统作为消费者，这多个合在一起就是一个 `Consumer Group` 消费者组，它们一般消费相同的消息。
- `Topic` 主题： 代表一类消息，比如订单消息，物流消息等等

在 `RocketMQ` 中，**一个队列只会被一个消费者消费**

## [分布式事务](https://snailclimb.gitee.io/javaguide/#/docs/system-design/data-communication/RocketMQ?id=分布式事务)

在 `RocketMQ` 中使用的是 **事务消息加上事务反查机制** 来解决分布式事务问题的。

![img](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/16ef38798d7a987f.png)

