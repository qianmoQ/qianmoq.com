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

### Jvm配置

---

- 编辑 **jvm.conf** 配置文件

```bash
vim conf/druid/broker/jvm.config
```

在文件中写入以下内容:

```bash
-server
-Xms3g
-Xmx3g
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.io.tmpdir=var/tmp
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Dderby.stream.error.file=var/druid/derby.log
```

- **-Xms**: 设置初始的(最小的)Heap的大小 此值可以设置与`-Xmx`相同，以避免每次垃圾回收完成后JVM重新分配内存
- **-Xmx**: 设置最大Heap的大小
- **-Duser.timezone**: 时区类型
- **-Dfile.encoding**: 文件编码
- **-Djava.io.tmpdir**: 系统缓冲临时目录
- **-Djava.util.logging.manager**: Log监控管理工具类
- **-Dderby.stream.error.file**: 内置数据库出现错误写入的文件

### 节点配置

---

```java
vim conf/druid/coordinator/runtime.properties
```

修改为以下内容

```java
druid.service=druid/coordinator
druid.port=8081
 
druid.coordinator.startDelay=PT30S
druid.coordinator.period=PT30S
```

- **druid.service**: 服务名称和`_common`中相关联
- **druid.port**: 当前服务端口
- **druid.coordinator.startDelay**: 延迟协调数据载入，这种延迟是一种破解，让它有足够的时间相信它拥有所有数据。
- **druid.coordinator.period**: 协调器的运行期。协调器通过在存储器中维护当前状态并定期查看可用的段集和服务的段来进行操作，以决定是否需要对数据拓扑进行任何更改。此属性设置每次运行之间的延迟。

### 命令行

---

!!! warning

    **服务启动命令脚本(进入druid-io安装文件的根目录)**

- 基本启动方式

```bash
java `cat conf/druid/coordinator/jvm.config | xargs` -cp "conf/druid/_common:conf/druid/coordinator:lib/*" io.druid.cli.Main server coordinator
```

- 后台启动方式

```bash
nohup java `cat conf/druid/coordinator/jvm.config | xargs` -cp "conf/druid/_common:conf/druid/coordinator:lib/*" io.druid.cli.Main server coordinator >coordinator.log 2>&1 &
```
