Apache Druid 是一个支持现代分析应用程序的实时数据库

### Druid特性

---

=== "构建快速"

    Druid 专为快速即席分析、即时数据可见性或支持高并发性很重要的工作流而设计。因此，Druid 通常用于为需要交互式、一致的用户体验的 UI 提供支持。

=== "数据集成"

    Druid 从Kafka和Amazon Kinesis等消息队列传输数据，并从HDFS和Amazon S3等批量加载文件。Druid 支持结构化和半结构化数据的大多数文件格式。

=== "广泛适用性"

    Druid为点击流、APM、供应链、网络遥测、数字营销、风险/欺诈和许多其他类型的数据解锁了新型查询和工作流程。Druid 专为对实时和历史数据进行快速、即席查询而构建。

=== "混合部署"

    Druid 可以部署在商用硬件上的任何`*NIX`环境中，无论是在云中还是在本地。部署 Druid 很简单：扩展和缩减就像添加和删除 Druid 服务一样简单。

### 文章分类

---

=== "Action"

    - [x] [common全局配置](./action/configuration_common.md)
        - [x] [common全局配置(MySQL)](./action/configuration_common_mysql.md)
        - [x] [common全局配置(HDFS)](./action/configuration_common_hdfs.md)
    - [x] [Broker节点配置部署](./action/configuration_node_broker.md)
    - [x] [Coordinator节点配置部署](./action/configuration_node_coordinator.md)
