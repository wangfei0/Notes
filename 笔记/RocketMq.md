# 一、设计理念
基于发布订阅模式。

**核心功能是：** 消息发送、消息存储（Broker）、消息消费。

以自研NameServer为注册中心，且注册中心集群之间互不通信，降低了实现复杂度，降低网络要求，性能较Zookeeper极大提高。

高效的IO存储机制。消息存储文件设计成文件组的概念，组内单个文件大小固定。使用内存映射机制，所有主题的消息存储基于顺序写。

消息至少被消费一次，允许消息被多次消费，由消费端实现幂等性。
# 二、设计目标
1. 架构模式
2. 顺序消息
3. 消息过滤
4. 消息存储
5. 消息高可用性
6. 消息到达（消费）低延迟
7. 确保消息必须被消费一次
8. 回溯消息
9. 消息堆积
10. 定时消息
11. 消息重试机制

# 三、NameServer路由中心
基于主题的订阅发布机制。

主要作用是为消息生产者和消息消费者提供关于主题Topic的路由信息。

Broker消息服务器在启动时向所有NameServer注册。消息生产者在发送消息之前，先从NameServer获取Broker服务器地址，然后根据负载均衡算法选择其中一台消息服务器进行消息发送。

NameServer与每台Broker服务器保持长连接。

NameServer集群中的每台服务器互不通信。

## NamerServer启动流程

1. 解析配置文件

- NameServerConfig


- NettyServerConfig

2. 创建NamesrvController实例

加载KV配置，创建NettyServer网络处理对象，开启两个定时任务（心跳检测）

- 定时任务1：NameServer每隔10s扫描一次Broker，移除处于不激活状态的Broker。
- 定时任务2：NameServer每隔10分钟打印一次KV配置。


3. 注册JVM钩子函数并启动服务器，以监听Broker、消息生产者的网络请求


## NamerServer路由注册、故障剔除

### 路由元信息
NameServer的路由实现类是：org/apache/rocketmq/namesrv/routeinfo/RouteInfoManager.java

该实现类引入了以下几个hashMap用于存储信息：
1. topicQueueTable：

Topic消息队列路由信息。存储的key为String类型的topic，value是List<QueueDate>类型。

其中QueueDate的结构如下：
- private String brokerName
- private int readQueueNums
- private int writeQueueNums
- private int perm
- private int topicSynFlag


2. brokerAddrTable

存储Broker的基础信息，key为String类型的brokerName，value为BrokerData，BrokerData结构如下：
- private String cluster    集群名
- private String brokerName
- private HashMap brokerAddrs

brokerAddrs是一个HahsMap，其key为Long类型brokerId，value为String类型broker address，==当brokerId为0时为集群中的Master，大于0表示为集群中的Slave。==

3. clusterAddrTable

Broker集群信息，存储集群中所有Broker名称。key为String类型的clusterName，value为Set<String /* brokerName */>

4. brokerLiveTable

Broker状态信息。NameServer每次收到心跳包时会替换该信息。key为String类型的brokerAddr，value为BrokerLiveInfo,BrokerLiveInfo结构为：

- private long lastUpdateTimestamp  存储上次收到Broker心跳包的时间
- private DataVersion dataVersion
- private Channel channel
- private String haServerAddr

5. filterServerTable
   Broker上的FilterServer列表，用于类模式消息过滤。key为String类型的brokerAddr，value为List<String>类型的Filter Server。


### 路由注册
RocketMQ的路由注册是通过Broke与NameServer的心跳功能实现的。

#### 1. Broker发送心跳包

#### 2. NameServer处理心跳包

> 注意：NameServer与Broker保持长连接，Broker状态存储在brokerLiveTable中，NameServer每收到一个心跳包，将更新brokerLiveTable中关于Broker的状态信息以及路由表（topicQueueTable，brokerAddrTable，brokerLiveTable，filterServerTable）。更新上述路由表（HashTable）使用了锁粒度较少的==读写锁==，允许多个消息发送者（Producer）并发读，保证消息发送时的高并发。


### 路由删除

Broker每30s向NameServer发送一个心跳包，心跳包包含BrokerId,broker地址,Broker名称,Broker所属集群名称，Broker关联的FilterServer列表。

NameServer每隔10s扫描brokerLiveTable状态表，如果BrokerLive的lastUpdateTimestamp的时间戳距当前时间超过120s，则认为Broker失效，移除该Broker，关闭与Broker的连接，同时更新topicQueueTable，brokerAddrTable，brokerLiveTable，filterServerTable。

由RouteInfoManaher#scanNotActiveBroker和onChannelDestroy两个方法完成，scanNotActiveBroker()每10s执行一次，遇到失效的broker（在BrokerLiveInfo中的lastUpdate  Timestamp的时间距离当前时间超过120s则失效）调用onChannelDestroy()方法删除。

### 路由发现

RocketMQ的路由发现不是实时的。由客户端拉去最新的路由，NameServer不会主动推送最新路由。

路由发现类为org/apache/rocketmq/namesrv/processor/DefaultRequestProcessor.java#getRouteInfoByTopic()

## 四、RocketMQ消息发送

### 消息发送方式
发送普通消息有三种方式：
1. 可靠同步发送：发送者向MQ执行发送消息API时，同步等待，直到服务器返回发送结果。
2. 可靠异步发送：发送者向MQ执行发送消息API时，指定消息发送成功后的回调函数，然后立即返回，消息发送者线程不阻塞。消息发送成功或失败的回调任务在一个新线程中执行。
3. 单向发送：发送者向MQ执行发送消息API时，直接返回，也不指定消息发送成功后的回调函数。

### RocketMQ的消息封装类

RocketMQ的消息封装类是：org/apache/rocketmq/common/message/Message.java

类属性如下：

```java
    private String topic;                       消息所属主题topic
    private int flag;                           消息Flag
    private Map<String, String> properties;     扩展属性
    private byte[] body;                        消息体
    private String transactionId;
```

properties中存储扩展属性，包括：
- tag：消息TAG，用于消息过滤。
- keys：Message索引键，多个用空格隔开，可以根据这些key快速索引到消息。
- waitStoreMsgOk：消息发送时是否等消息存储完成后再返回。
- delayTimeLevel：消息延迟级别，用于定时消息或消息重试。


### 生产者启动流程
#### DefaultMQProducer消息发送者
DefaultMQProducer是默认的消息生产者实现类。实现了MQAdmin接口。

#### 消息生产者启动流程

从DefaultMQProducerImpl类的start()方法开始：

1. 检查 producerGroup 是否符合要求，并该改变生产者的instaneName为进程ID。
2. 创建MQClientInstance实例，整个JVM实例中只存在一个MQClientManager实例，维护一个MQClientInstance缓存表ConcurrentMap<String/* clientId */, MQClientInstance> factoryTable = new ConcurrentHashMap<String, MQClientInstance>()。即同一个clientId只会创建一个MQClientInstance。
3. 向MQClientInstance注册，讲当前生产者加入到MQClientInstance管理中，方便后续调用网络请求，进行心跳检测等。
4. 启动MQClientInstance，如果MQClientInstancei已经启动，则本此启动不会真正执行。

> clientId格式为客户端IP@instanceName(@unitName可选)。在类ClientConfig#buildMClientId()中创建。

在上一步中改变生产者的instaneName为进程ID目的即避免不同进程的相互影响，放置linetId相同引起混乱。

### 消息发送基本流程

1. 验证消息
2. 查找路由
3. 消息发送


默认消息发送以同步方式发送，默认超时时间为3s。

#### 消息长度验证

规则是主题名称、消息体不能为空、消息长度不能等于且默认不能超过允许发送消息的最大长度4M(maxMessageSize=1024x1024x4)。

#### 查找主题路由信息

对应方法为DefaultMQProducerImpl#tryToFindTopicPublishInfo()。该方法返回一个TopicPublishInfo。

TopicPublishInfo的类结构为：

```java
private boolean orderTopic = false;     是否是顺序消息
private boolean haveTopicRouterInfo = false;        
private List<MessageQueue> messageQueueList = new ArrayList<MessageQueue>();    该主题队列的消息队列
private volatile ThreadLocalIndex sendWhichQueue = new ThreadLocalIndex();  每选择一次消息队列，该值自增1，用于消息队列选择
private TopicRouteData topicRouteData;
```
```java 
private String orderTopicConf;
private List<QueueData> queueDatas;     topic队列元数据
private List<BrokerData> brokerDatas;       topic分布的broker元数据
private HashMap<String/* brokerAddr */, List<String>/* Filter Server */> filterServerTable;  broker上过滤服务器地址列表
```
#### 选择消息队列

根据路由信息选择消息队列。返回的消息队列按照broker、序号排序。

1. 默认机制

调用TopicPublishInfo#selectOneMessageQueue()方法

第一次调用直接递增，第二次就会规避之前调用失败的Broker

2. Broker故障延迟机制

实现方法为MQFaultStrategy#selectOneMessageQueue。

#### 消息发送

DefaultMQProducerImpl#sendKernelImpl(final Message msg,
final MessageQueue mq,
final CommunicationMode communicationMode,
final SendCallback sendCallback,
final TopicPublishInfo topicPublishInfo,
final long timeout)消息发送API核心入口。

方法参数：
1. final Message msg：待发送消息
2. final MessageQueue mq：消息将发送到该消息队列上
3. final CommunicationMode communicationMode：消息发送模式，sync、async、oneway
4. final SendCallback sendCallback：异步消息回调函数
5. final TopicPublishInfo topicPublishInfo：主题路由信息
6. final long timeout：消息发送超时时间


### 批量消息发送

批量消息发送是将同一主题的多条消息一起打包发送到消息服务端，减少网络调用次数，提高网络传输效率。

单批次消息发送总长度不能超过DefaultMQProducer#maxMessageSize。

对单条消息内容采用固定格式进行存储，发送端将一批消息封装为MessageBatch对象，MessageBatch对象内部持有List<Message> messages。MessageBatch只需要将该集合中的每条消息的消息体聚合为一个byte[]数值。

RocketMQ的网络请求类设计为：

RemotingCommand
```java 
    private int code;
    private LanguageCode language = LanguageCode.JAVA;
    private int version = 0;
    private int opaque = requestId.getAndIncrement();
    private int flag = 0;
    private String remark;
    private HashMap<String, String> extFields;
    private transient CommandCustomHeader customHeader;

    private SerializeType serializeTypeCurrentRPC = serializeTypeConfigInThisServer;

    private transient byte[] body;      消息体
```

在创建RemotingCommand对象时调用MessageBatch#encode()方法填充到RemotingCommand的body域中。

消息发送过程同单条消息发送相同。

