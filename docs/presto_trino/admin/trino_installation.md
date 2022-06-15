---
template: overrides/main.html
---

!!! warning "重要提示"

    安装 Admin 后,各个节点需要打通 ssh 免密登录, 如果未打通 ssh 免密登录的情况下需要将连接私钥上传到 Admin 节点相应的位置

#### 下载相关安装包

---

Admin安装包文件一般为`<version>.tar.gz`格式.

```bash
wget https://github.com/trinodb/trino-admin/archive/2.5.tar.gz
```

下载 Presto 安装包

```bash
wget https://repo1.maven.org/maven2/io/prestosql/presto-server-rpm/341/presto-server-rpm-341.rpm
```

#### 解压相关软件

---

解压压缩包, 并将解压后的文件夹放到当前目录下.

```bash
tar -zxvf 2.5.tar.gz
```

进入解压后的文件夹

```bash
cd trino-admin-2.5/
```

#### 构建 Python 环境

---

构建 Python 虚拟环境

```bash
virtualenv trino-admin
```

激活 Python 虚拟环境

```bash
source trino-admin/bin/activate
```

更新相关 Python 组件

```bash
pip install --upgrade pip
```

安装 Admin 相关依赖

```bash
pip install -r requirements.txt
```

!!! note "注意"

    下载依赖根据当前客户端网络而定,也可以使用相关国内源.

!!! warning "重要提示"

    Presto 的 RPM 安装包需要校验 OpenJDK 版本, 可以使用以下命令安装
    ```bash
    yum install java-11-openjdk
    ```
    需要注意的是 JDK 需要在每个节点中进行安装, 或者使用 `presto-admin package install xxx` 方式安装.

我们可以使用以下命令测试安装是否成功

```bash
presto-admin --version
```

返回类似以下的版本号标记安装成功

```bash
presto-admin 2.5
```
