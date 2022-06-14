---
template: overrides/main.html
tags:
  - Apache
  - Druid
  - 大数据
---

!!! note

    以下操作需要进入druid-io安装文件的根目录下操作，比如我的druid-io安装目录为： `/hadoop/dc/druid-0.9.1.1`
    那么我们需要操作 `cd /hadoop/dc/druid-0.9.1.1`

### 基本配置

---

- common.runtime.properties配置文件

```bash
vim conf/druid/_common/common.runtime.properties
```

修改为以下内容:

```bash
#
# Licensed to Metamarkets Group Inc. (Metamarkets) under one
# or more contributor license agreements. See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership. Metamarkets licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License. You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied. See the License for the
# specific language governing permissions and limitations
# under the License.
#
 
#
# Extensions
#
 
# This is not the full list of Druid extensions, but common ones that people often use. You may need to change this list
# based on your particular setup.
# druid.extensions.loadList=["druid-kafka-eight", "druid-s3-extensions", "druid-histogram", "druid-datasketches", "druid-lookups-cached-global", "mysql-metadata-storage"]
druid.extensions.loadList=["druid-kafka-eight", "druid-s3-extensions", "druid-histogram", "druid-datasketches", "druid-lookups-cached-global"]
 
# If you have a different version of Hadoop, place your Hadoop client jar files in your hadoop-dependencies directory
# and uncomment the line below to point to your directory.
#druid.extensions.hadoopDependenciesDir=/my/dir/hadoop-dependencies
 
#
# Logging
#
 
# Log all runtime properties on startup. Disable to avoid logging properties on startup:
druid.startup.logging.logProperties=true
 
#
# Zookeeper
#
 
druid.zk.service.host=zk.host.ip
druid.zk.paths.base=/druid
 
#
# Metadata storage
#
 
# For Derby server on your Druid Coordinator (only viable in a cluster with a single Coordinator, no fail-over):
druid.metadata.storage.type=derby
druid.metadata.storage.connector.connectURI=jdbc:derby://metadata.store.ip:1527/var/druid/metadata.db;create=true
druid.metadata.storage.connector.host=metadata.store.ip
druid.metadata.storage.connector.port=1527
 
# For MySQL:
#druid.metadata.storage.type=mysql
#druid.metadata.storage.connector.connectURI=jdbc:mysql://db.example.com:3306/druid
#druid.metadata.storage.connector.user=...
#druid.metadata.storage.connector.password=...
 
# For PostgreSQL (make sure to additionally include the Postgres extension):
#druid.metadata.storage.type=postgresql
#druid.metadata.storage.connector.connectURI=jdbc:postgresql://db.example.com:5432/druid
#druid.metadata.storage.connector.user=...
#druid.metadata.storage.connector.password=...
 
#
# Deep storage
#
 
# For local disk (only viable in a cluster if this is a network mount):
druid.storage.type=local
druid.storage.storageDirectory=var/druid/segments
 
# For HDFS (make sure to include the HDFS extension and that your Hadoop config files in the cp):
#druid.storage.type=hdfs
#druid.storage.storageDirectory=/druid/segments
 
# For S3:
#druid.storage.type=s3
#druid.storage.bucket=your-bucket
#druid.storage.baseKey=druid/segments
#druid.s3.accessKey=...
#druid.s3.secretKey=...
 
#
# Indexing service logs
#
 
# For local disk (only viable in a cluster if this is a network mount):
druid.indexer.logs.type=file
druid.indexer.logs.directory=var/druid/indexing-logs
 
# For HDFS (make sure to include the HDFS extension and that your Hadoop config files in the cp):
#druid.indexer.logs.type=hdfs
#druid.indexer.logs.directory=/druid/indexing-logs
 
# For S3:
#druid.indexer.logs.type=s3
#druid.indexer.logs.s3Bucket=your-bucket
#druid.indexer.logs.s3Prefix=druid/indexing-logs
 
#
# Service discovery
#
 
druid.selectors.indexing.serviceName=druid/overlord
druid.selectors.coordinator.serviceName=druid/coordinator
 
#
# Monitoring
#
 
druid.monitoring.monitors=["com.metamx.metrics.JvmMonitor"]
druid.emitter=logging
druid.emitter.logging.logLevel=info
```

* [ ] **druid.extensions.loadList:** 该列表加载的依赖需要放置到 **<druid-io根目录>/extensions/** 下面加载的依赖名为文件夹名称
* [ ] **druid.zk.service.host && druid.zk.paths.base:** 对于zookeeper的集群主机配置&&在zookeeper中的根路径配置
* [ ] **druid.metadata.storage.*:** 对于druid-io的存储介质配置默认为内置的derby数据库,对于配置mysql存储介质的话需要提前将mysql链接jar放置到 **<druid-io根目录>/lib/** 目录下
* [ ] **druid.storage.type && druid.storage.storageDirectory:** 对于druid-io生成的segments保存位置默认为local && segments保存的文件夹路径(可以为 **local**，**hdfs**，**s3**)
* [ ] **druid.indexer.logs.*:** 对于druid-io的索引日志配置默认为local(可以为 **local**，**hdfs**，**s3**)
* [ ] **druid.selectors.indexing.serviceName:** 对于druid-io的索引服务名称
* [ ] **druid.selectors.coordinator.serviceName:** 对于druid-io的coordinator服务名称
* [ ] **druid.monitoring.monitors:** 对于druid-io的监控指标配置，配置为列表
* [ ] **druid.emitter:对于druid-io的插件发射器配置比如log日志配置为:**

```bash
druid.emitter=logging
druid.emitter.logging.logLevel=info
```

### 日志配置

---

- log4j2.xml配置文件

```bash
vim conf/druid/_common/log4j2.xml
```

修改为以下内容:

```bash
<?xml version="1.0" encoding="UTF-8" ?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{ISO8601} %p [%t] %c - %m%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

该文件详细配置可参考[log4j](https://logging.apache.org/log4j/2.x/)官网进行
