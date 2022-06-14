---
template: overrides/main.html
tags:
- ClickHouse
- OLAP
- 大数据
- Table Engine
- Kafka
- Issues
---

线上ClickHouse集群发现Kafka引擎表不能正常工作，物化视图未触发，无法将数据正常的摄取到ClickHouse本地表中,查看ClickHouse日志(`clickhouse-server.err.log`)发现Kafka引擎表出现poll失败退出的问题,日志中显示如下内容

```java
2022.03.09 16:49:11.188012 [ 538056 ] {de2d06ca-8b96-4ad7-b805-09414f05fdad} <Error> StorageKafka (kafka_engine_table): Error during draining: Local: Maximum application poll interval (max.poll.interval.ms) exceeded
2022.03.09 16:49:11.289315 [ 538056 ] {de2d06ca-8b96-4ad7-b805-09414f05fdad} <Error> StorageKafka (kafka_engine_table): Error during draining: Local: Maximum application poll interval (max.poll.interval.ms) exceeded
2022.03.09 16:55:40.776456 [ 8094 ] {} <Warning> StorageKafka (kafka_engine_table): [rdk:MAXPOLL] [thrd:main]: Application maximum poll interval (300000ms) exceeded by 51ms (adjust max.poll.interval.ms for long-running message processing): leaving group
2022.03.09 16:55:45.277079 [ 8100 ] {} <Warning> StorageKafka (kafka_engine_table): [rdk:MAXPOLL] [thrd:main]: Application maximum poll interval (300000ms) exceeded by 50ms (adjust max.poll.interval.ms for long-running message processing): leaving group
```

#### 问题排查

---

Kafka引擎表在poll数据的时候直接超时并离开消费组. 但是在集群中有大量的Kafka引擎表都可以正常摄取数据,我们首先排除了这是网络的问题.

期间也尝试过删除Kafka引擎表重建还是同样的问题, 直接查询Kafka引擎表可以正常获取数据 `select * from kafka_engine_table_name limit 100` 排除是Kafka服务的问题.

查看了以下我们的ClickHouse版本是 `20.11.3.3`, 经过翻阅ClickHouse源码发现 `https://github.com/ClickHouse/ClickHouse/blob/master/src/Core/Settings.h` 配置中有一个名为 `background_schedule_pool_size` 的配置选项.

源码解释如下:

!!! note

    M(UInt64, background_schedule_pool_size, 128, "Number of threads performing background tasks for replicated tables, dns cache updates. Only has meaning at server startup.", 0) 

改参数用于配置表后台运行task的选项,初步判断是这里的问题,但是看源码这里显示的是默认值 `128` 理论上我们的集群中是不会超过这个值的, 我们前往ClickHouse服务器中进行了参数的查询发现系统配置的是 `16`

然后结合该文件的提交历史发现 `alesapin 2021/6/8, 18:55 Increase background schedule pool default size` 提交历史的[commit](https://github.com/ClickHouse/ClickHouse/commit/5736eaf8fd5b73a7a5f7d38ea07b73fd77a0623e) 
然后结合发布历史版本发现改参数是在 `21.3.9.83` 版本以后才设置的默认值 `128`

我们通过以下SQL查询ClickHouse服务器的默认配置和监控指标:

```sql
WITH t0 AS (
  SELECT
    value AS defaultValue
  FROM
    `system`.settings
  WHERE
    name = 'background_schedule_pool_size'
),
t1 AS (
  SELECT
    value AS currentValue
  FROM
    `system`.metrics
  WHERE
    metric = 'BackgroundSchedulePoolTask'
)
SELECT
  defaultValue,
  currentValue
FROM
  t0,
  t1
```

得到以下结果

```bash
Query id: 82dfca06-b416-49c8-8a96-5d30b2abebc2

┌─defaultValue─┬─currentValue─┐
│ 16           │          16  │
└──────────────┴──────────────┘

1 rows in set. Elapsed: 0.003 sec.
```

发现当前值已经达到配置的最大值，需要增加以扩展后台执行的Task任务数, 我们参照新源码的 `128` 配置. 

!!! warning

    根据源码解释说该参数需要重启服务,配置完成后重启ClickHouse集群.

#### 配置方式

---

修改 `users.xml` 增加如下配置

```xml
<background_schedule_pool_size>128</background_schedule_pool_size>
```

!!! warning

    增加到 `profiles   --->   default` 节点下

重启ClickHouse集群后,Kafka引擎表可以正常摄取数据!!!
