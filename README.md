## 为什么要抛弃oracle

图表和报表需求的日益增多，带来了巨大的数据固化工作，数十张固化表和存储过程的开发维护成本太高，不如直接换个分析性能更好的数据库

## 数据库选型

一些常见的分析型数据库：

|数据库|是否开源|数据库架构|问题|
|---|---|---|---|
|Vertica|商业（有社区版）|列存储MPP架构|收费，社区版有数据大小限制|
|InfoBright|商业（有社区版）|列存储|收费，社区版没有备份和扩展能力|
|TeraData|商业||收费，用的人少|
|Palo|商业|百度云存储|收费|
|RedShift|商业|亚马逊云存储|收费|
|Impala|开源|基于hadoop的分布式查询框架|部署成本、学习成本高，性能一般|
|Spark SQL|开源|基于hadoop的分布式查询框架|部署成本、学习成本高，性能一般|
|Greenplum|开源|列存储MPP架构|部署成本、学习成本相对低，性能一般|
|MonetDB|开源|列存储MPP架构|部署成本、学习成本相对低，性能好，稳定性差|
|Druid|开源|离线预聚合|部署成本、学习成本高，性能好|
|Kylin|开源|离线预聚合|部署成本、学习成本高，性能好|
|Clickhouse|开源|列存储MPP架构|部署成本、学习成本相对低，性能好|
|hive|开源|基于hadoop的分析型数据库|部署成本、学习成本高，性能一般|
|presto|开源|基于hadoop的分布式查询框架|部署成本、学习成本高，性能一般|

**总结一下**：开源的慢，收费的贵，离线预聚合架构虽然很快但是基于hadoop，学习部署成本相对高同时还需要对一些表重新设计。所以最后选用了clickhouse--开源且快，而且经受了生产环境的考验（俄罗斯搜索引擎：www.yandex.com,360个节点）

## clickhouse优缺点

### 优点

- 开源
- 基准测试中表现优异， 速度是传统数据库1000倍，Vertica5倍，monetdb20倍
- 在开源数据库中效率最高，跟monetdb比较速度更快而且经受了生产环境的考验（俄罗斯搜索引擎：www.yandex.com,360个节点）

### 缺点

- 不支持update和delete，所以要结合自己的业务场景
- clickhouse不完全支持标准sql，有些语法需要去官方文档查看
- 文档只有英文
- 搜索引擎中的解决方案少
- 数据导入比较麻烦，我们现在是用kattle从oracle导出成csv然后执行linux服务器中的bash命令调用clickhouse的命令行接口导入数据

## clickhouse常用链接

### [官网](https://clickhouse.yandex/)

### [各类数据库之间基准测试结果比较](https://clickhouse.yandex/benchmark.html)

### [官方文档](https://clickhouse.yandex/docs/en/single)

## 安装

### docker安装

- [clickhouse-server](https://hub.docker.com/r/yandex/clickhouse-server/)
- [clickhouse-client](https://hub.docker.com/r/yandex/clickhouse-client/)

注意：配置文件和数据目录最好挂载到服务器目录

### Debian/Ubuntu在线安装

```bash
deb http://repo.yandex.ru/clickhouse/deb/stable/ main/
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv E0C56BD4    # optional
sudo apt-get update
sudo apt-get install clickhouse-client clickhouse-server
```

离线安装可以去github上下载源代码编译安装

### CentOS/RHEL离线安装

[在此处](https://packagecloud.io/altinity/clickhouse)下载rpm包

### 图形界面的客户端工具可以使用[DBeaver](https://dbeaver.jkiss.org/download/)

## 配置文件

1. /etc/clickhouse-server/config.xml 端口配置、本地机器名配置、内存设置等 

```xml
<!-- 有必要了解下的配置，切记不要直接复制粘贴，这个不全 -->
<?xml version="1.0"?>
<yandex>
   <!-- 日志 -->
    <logger>
        <level>trace</level>
        <log>/data1/clickhouse/log/server.log</log>
        <errorlog>/data1/clickhouse/log/error.log</errorlog>
        <size>1000M</size>
        <count>10</count>
    </logger>
    <!-- 打开此配置才可以接收远程的数据库链接 -->
    <listen_host>::</listen_host>
    <!-- 不支持ipv6用这个 -->
    <!-- <listen_host>0.0.0.0</listen_host> -->
    <!-- 支持ipv6用下面这两句话 -->
    <listen_host>::1</listen_host>
    <listen_host>127.0.0.1</listen_host>
    <!-- 端口 -->
    <http_port>8123</http_port>
    <tcp_port>9000</tcp_port>
    <interserver_http_port>9009</interserver_http_port>
    <!-- 存储路径 -->
    <path>/data1/clickhouse/</path>
    <tmp_path>/data1/clickhouse/tmp/</tmp_path>
    <!-- user配置 -->
    <users_config>users.xml</users_config>
    <default_profile>default</default_profile>
    <!-- 集群配置 -->
    <include_from>/data1/clickhouse/metrika.xml</include_from>
    <!-- 时区-docker安装需要配置这东西，要不然时区不对 -->
    <timezone>Asia/Shanghai</timezone>
    <!-- 集群部署时需要配置本机host -->
    <interserver_http_host>ora1</interserver_http_host>
</yandex>
```

2. /etc/clickhouse-server/users.xml 用户名密码

```xml
<?xml version="1.0"?>
<yandex>
    <profiles>
        <!-- 读写用户设置  -->
        <default>
            <max_memory_usage>10000000000</max_memory_usage>
            <use_uncompressed_cache>0</use_uncompressed_cache>
            <load_balancing>random</load_balancing>
        </default>

        <!-- 只写用户设置  -->
        <readonly>
            <max_memory_usage>10000000000</max_memory_usage>
            <use_uncompressed_cache>0</use_uncompressed_cache>
            <load_balancing>random</load_balancing>
            <readonly>1</readonly>
        </readonly>
    </profiles>

    <!-- 配额  -->
    <quotas>
        <!-- Name of quota. -->
        <default>
            <interval>
                <duration>3600</duration>
                <queries>0</queries>
                <errors>0</errors>
                <result_rows>0</result_rows>
                <read_rows>0</read_rows>
                <execution_time>0</execution_time>
            </interval>
        </default>
    </quotas>

    <users>
        <!-- 读写用户  -->
        <default>
            <!-- 密码为空  -->
            <password_sha256_hex>967f3bf355dddfabfca1c9f5cab39352b2ec1cd0b05f9e1e6b8f629705fe7d6e</password_sha256_hex>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
        </default>

        <!-- 只读用户  -->
        <ck>
            <password_sha256_hex>967f3bf355dddfabfca1c9f5cab39352b2ec1cd0b05f9e1e6b8f629705fe7d6e</password_sha256_hex>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>readonly</profile>
            <quota>default</quota>
        </ck>
    </users>
</yandex>
```
3. /etc/users.xml 集群配置

```xml
<yandex>
    <clickhouse_remote_servers>
        <hma3s3r>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>ora1</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <replica>
                    <internal_replication>true</internal_replication>
                    <host>ora2</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>ora3</host>
                    <port>9000</port>
                </replica>
            </shard>
        </hma3s3r>
    </clickhouse_remote_servers>

    <zookeeper-servers>
    <node index="1">
        <host>ora1</host>
        <port>2181</port>
    </node>
    <node index="2">
        <host>ora2</host>
        <port>2181</port>
    </node>
    <node index="3">
        <host>ora3</host>
        <port>2181</port>
    </node>
    </zookeeper-servers>

    <macros>
        <replica>ora2</replica>
    </macros>

    <networks>
    <ip>::/0</ip>
    </networks>

    <clickhouse_compression>
        <case>
        <min_part_size>10000000000</min_part_size>
        <min_part_size_ratio>0.01</min_part_size_ratio>
        <method>lz4</method>
        </case>
    </clickhouse_compression>
</yandex>
```

4. zookeeper也要配置，默认配置基本不能用，配置zoo.cfg如下：

```
clientPort=2181
dataDir=/data
dataLogDir=/datalog
tickTime=2000
initLimit=30000
syncLimit=2
maxClientCnxns=60
maxClientCnxns=2000
maxSessionTimeout=60000000
autopurge.snapRetainCount=10
autopurge.purgeInterval=1
preAllocSize=131072
snapCount=3000000
leaderServes=yes
```

## 启动服务

server

- docker:docker run -d --name clickhouse-server --ulimit nofile=262144:262144 -p 8123:8123 -p 9000:9000 yandex/clickhouse-server
- nohup clickhouse-server --config-file=/etc/clickhouse-server/config.xml > null 2>&1 &

client

- docker:docker run -it --name clickhouse-client --rm --net=host yandex/clickhouse-client
- sh:clickhouse-client -h 127.0.0.1

##  测试

### 测试用sql

```sql
-- 数据量 ka02:24w,kd01:3k,kb01:1k,kd37:300w,kd44:7亿
SELECT
    akb020,
    akb021,
    akc264,
    multiIf(0 = akc264_, 0, round((akc264 - akc264_) / akc264_, 4) * 100) AS akc264_comparde,
    bkd248,
    multiIf(0 = bkd248_, 0, round((bkd248 - bkd248_) / bkd248_, 4) * 100) AS bkd248_comparde,
    aae019,
    multiIf(0 = aae019_, 0, round((aae019 - aae019_) / aae019_, 4) * 100) AS aae019_comparde,
    person_count,
    multiIf(0 = person_count_, 0, round((person_count - person_count_) / person_count_, 4) * 100) AS person_count_comparde,
    avg_fee,
    multiIf(0 = avg_fee_, 0, round((avg_fee - avg_fee_) / avg_fee_, 4) * 100) AS avg_fee_comparde,
    rate,
    rate - rate_ AS rate_comparde,
    aae019_akc264,
    aae019_akc264 - aae019_akc264_ AS aae019_akc264_comparde,
    ake061,
    ake061 - ake061_ AS ake061_comparde,
    charge_in,
    multiIf(0 = charge_in_, 0, round((charge_in - charge_in_) / charge_in_, 4) * 100) AS charge_in_comparde,
    charge_out,
    multiIf(0 = charge_out_, 0, round((charge_out - charge_out_) / charge_out_, 4) * 100) AS charge_out_comparde,
    charge_in_scale,
    charge_in_scale - charge_in_scale_ AS charge_in_scale_comparde,
    content_in,
    multiIf(0 = content_in_, 0, round((content_in - content_in_) / content_in_, 4) * 100) AS content_in_comparde,
    content_out,
    multiIf(0 = content_out_, 0, round((content_out - content_out_) / content_out_, 4) * 100) AS content_out_comparde,
    content_in_scale,
    content_in_scale - content_in_scale_ AS content_in_scale_comparde
FROM
(
    SELECT
        akb020,
        max(akb021) AS akb021,
        sum(akc264) AS akc264,
        count() AS bkd248,
        sum(pc) AS person_count,
        sum(_aae019) AS aae019,
        multiIf(0 = person_count, 0, round(aae019 / person_count, 2)) AS avg_fee,
        multiIf(0 = bkd248, 0, round(person_count / bkd248, 4) * 100) AS rate,
        multiIf(0 = akc264, 0, round(aae019 / akc264, 4) * 100) AS aae019_akc264,
        multiIf(0 = aae019, 0, round(sum(ake061) / aae019, 4) * 100) AS ake061,
        sum(bkd670 + bkd671) AS charge_in,
        sum(bkd672) AS charge_out,
        multiIf(0 = aae019, 0, round(charge_in / aae019, 4) * 100) AS charge_in_scale,
        sum(bkd670_c + bkd671_c) AS content_in,
        sum(bkd672_c) AS content_out,
        multiIf(0 = aae019, 0, round(content_in / aae019, 4) * 100) AS content_in_scale
    FROM
    (
        SELECT
            akb020,
            akb021,
            bka974,
            bke341,
            bkb005,
            bkb006,
            aka101,
            bkb007,
            akc190,
            aae140,
            bkc293,
            aka130,
            bka130,
            bkd001,
            bkd002,
            bkd003,
            bkd004,
            bkd005,
            bkd006,
            akc264,
            bke041,
            akc264,
            _aae019,
            akc228,
            akc268,
            ake051,
            ake061,
            bkd670,
            bkd671,
            bkd672,
            bkd670_c,
            bkd671_c,
            bkd672_c,
            bkc016,
            pc
        FROM
        (
            SELECT
                *,
                bkd001,
                bkd002,
                bkd003,
                bkd004,
                bkd006
            FROM
            (
                SELECT
                    akb020,
                    akb021,
                    bka974,
                    bke341,
                    bkb005,
                    bkb006,
                    aka101,
                    bkb007,
                    akc190,
                    aae072,
                    aae140,
                    bkc293,
                    aka130,
                    bka130,
                    bkd005,
                    akc264,
                    bke041,
                    bke042,
                    bke057,
                    bkc016
                FROM kd37
                ANY LEFT JOIN kb01 USING (akb020)
                WHERE (bkc016 >= '2017-01-01') AND (bkc016 <= '2018-01-01')
            )
            ANY INNER JOIN kd01 USING (bkd005)
        )
        ANY LEFT JOIN
        (
            SELECT
                akb020,
                akc190,
                aae072,
                1 AS pc,
                sum(aae019) AS _aae019,
                sum(multiIf('1' = aka065, aae019, 0)) AS bkd670,
                sum(multiIf('2' = aka065, aae019, 0)) AS bkd671,
                sum(multiIf('3' = aka065, aae019, 0)) AS bkd672,
                sum(multiIf('1' = _aka065, aae019, 0)) AS bkd670_c,
                sum(multiIf('2' = _aka065, aae019, 0)) AS bkd671_c,
                sum(multiIf('3' = _aka065, aae019, 0)) AS bkd672_c,
                sum(akc228) AS akc228,
                sum(akc268) AS akc268,
                sum(ake051) AS ake051,
                ((_aae019 - akc228) - akc268) - ake051 AS ake061
            FROM
            (
                SELECT
                    akb020,
                    akc190,
                    aae072,
                    ake001,
                    aae019,
                    akc228,
                    akc268,
                    ake051,
                    aka065,
                    _aka065
                FROM kd44
                ANY INNER JOIN
                (
                    SELECT
                        aka060 AS ake001,
                        aka065 AS _aka065
                    FROM ka02
                ) USING (ake001)
                WHERE (bkc016 >= '2017-01-01') AND (bkc016 <= '2018-01-01')
            )
            GROUP BY
                akb020,
                akc190,
                aae072
        ) USING (akb020, akc190, aae072)
    )
    GROUP BY akb020
)
ANY LEFT JOIN
(
    SELECT
        akb020,
        sum(akc264) AS akc264_,
        count() AS bkd248_,
        sum(pc) AS person_count_,
        sum(_aae019) AS aae019_,
        multiIf(0 = person_count_, 0, round(aae019_ / person_count_, 2)) AS avg_fee_,
        multiIf(0 = bkd248_, 0, round(person_count_ / bkd248_, 4) * 100) AS rate_,
        multiIf(0 = akc264_, 0, round(aae019_ / akc264_, 4) * 100) AS aae019_akc264_,
        multiIf(0 = aae019_, 0, round(sum(ake061) / aae019_, 4) * 100) AS ake061_,
        sum(bkd670 + bkd671) AS charge_in_,
        sum(bkd672) AS charge_out_,
        multiIf(0 = aae019_, 0, round(charge_in_ / aae019_, 4) * 100) AS charge_in_scale_,
        sum(bkd670_c + bkd671_c) AS content_in_,
        sum(bkd672_c) AS content_out_,
        multiIf(0 = aae019_, 0, round(content_in_ / aae019_, 4) * 100) AS content_in_scale_
    FROM
    (
        SELECT
            akb020,
            akb021,
            bka974,
            bke341,
            bkb005,
            bkb006,
            aka101,
            bkb007,
            akc190,
            aae140,
            bkc293,
            aka130,
            bka130,
            bkd001,
            bkd002,
            bkd003,
            bkd004,
            bkd005,
            bkd006,
            akc264,
            bke041,
            akc264,
            _aae019,
            akc228,
            akc268,
            ake051,
            ake061,
            bkd670,
            bkd671,
            bkd672,
            bkd670_c,
            bkd671_c,
            bkd672_c,
            bkc016,
            pc
        FROM
        (
            SELECT
                *,
                bkd001,
                bkd002,
                bkd003,
                bkd004,
                bkd006
            FROM
            (
                SELECT
                    akb020,
                    akb021,
                    bka974,
                    bke341,
                    bkb005,
                    bkb006,
                    aka101,
                    bkb007,
                    akc190,
                    aae072,
                    aae140,
                    bkc293,
                    aka130,
                    bka130,
                    bkd005,
                    akc264,
                    bke041,
                    bke042,
                    bke057,
                    bkc016
                FROM kd37
                ANY LEFT JOIN kb01 USING (akb020)
                WHERE (bkc016 >= '2016-01-01') AND (bkc016 <= '2017-01-01')
            )
            ANY INNER JOIN kd01 USING (bkd005)
        )
        ANY LEFT JOIN
        (
            SELECT
                akb020,
                akc190,
                aae072,
                1 AS pc,
                sum(aae019) AS _aae019,
                sum(multiIf('1' = aka065, aae019, 0)) AS bkd670,
                sum(multiIf('2' = aka065, aae019, 0)) AS bkd671,
                sum(multiIf('3' = aka065, aae019, 0)) AS bkd672,
                sum(multiIf('1' = _aka065, aae019, 0)) AS bkd670_c,
                sum(multiIf('2' = _aka065, aae019, 0)) AS bkd671_c,
                sum(multiIf('3' = _aka065, aae019, 0)) AS bkd672_c,
                sum(akc228) AS akc228,
                sum(akc268) AS akc268,
                sum(ake051) AS ake051,
                ((_aae019 - akc228) - akc268) - ake051 AS ake061
            FROM
            (
                SELECT
                    akb020,
                    akc190,
                    aae072,
                    ake001,
                    aae019,
                    akc228,
                    akc268,
                    ake051,
                    aka065,
                    _aka065
                FROM kd44
                ANY INNER JOIN
                (
                    SELECT
                        aka060 AS ake001,
                        aka065 AS _aka065
                    FROM ka02
                ) USING (ake001)
                WHERE (bkc016 >= '2016-01-01') AND (bkc016 <= '2017-01-01')
            )
            GROUP BY
                akb020,
                akc190,
                aae072
        ) USING (akb020, akc190, aae072)
    )
    GROUP BY akb020
) USING (akb020)
```

### 测试结果

|服务器情况|用时|
|---|---|
|oracle|>30分钟|
|clickhouse单节点|32s|
|clickhouse多节点分三片|32s|
|clickhouse多节点分三片（sql优化后）|16s|

对于单表的sum，count性能分3片大概提高了2.5倍，测试的sql过长不在此一一列举

### 测试过程中发现需要注意的地方

1. 使用分布式建表语句建表，简单
2. 常用的表引擎是MergeTree，其他的分布式和复制表也是基于MergeTree表（MergeTree的关键在于以主键日期对数据进行分区，大数据量下以日期为查询条件是必要的，这个日期条件可以显著提高查询效率）
3. 尽量少用group by，很影响性能
4. 分布式的查询，使用子查询结果集过大十分影响性能，甚至于不如单节点的效率高(猜测是子查询的结果集会合并，消耗很大的网络io时间)。
5. 分布式表尽量只有一个，多个分布式表很消耗网络io（多个分布式表意味着有几个表需要提前从各个切片聚集数据）
6. 子查询结果集过大有时难以避免，这时候可能需要改表结构，但是又增加了开发和维护成本，要不然就直接双机互备，抛弃分布式的性能提升，这个东西根据自己的情况考量即可。

### 分布式的方案

此时要注意区分集群备份和表复制之间的区别

1. 最简单方案

MergeTree + Distributed

这种方案不安全

2. 高可用方案

ReplicatedMergeTree + Distributed + 集群复制（internal_replication=true）

节点宕机时由备机接管分片请求（没有备机主机概念，同一切片中互为备份，哪个坏了都不影响使用）

3. 高性能方案

ReplicatedMergeTree + Distributed

此方案一旦有节点宕机，分布式表立刻无法使用

**我们选择的是高性能方案**，一是我们项目的普遍客户对成本敏感，大多数只能提供2-3台服务器（有时候甚至只有一台），二是分析型的业务对服务器的稳定性要求不高，因此我们弃用了高可用选择了高性能方案
