# 前言
记录timescaledb的文档
分布式参考链接：
https://docs.timescale.com/timescaledb/latest/how-to-guides/multinode-timescaledb/
https://www.bookstack.cn/read/TimescaleDB-2.1-en/9e03c38f765e20fe.md

# 配置修改
## 1. pg配置
```
vim /etc/postgresql/14/main/pg_hba.conf 
host    all             all             0.0.0.0/0               trust  # 允许其他节点访问
```
### 1.1 pg配置 for benchmark配置
pg_hba.conf加密方式为sha256的都要改成md5验证
使用以下命令查询密码的认证方式
```
SELECT rolpassword FROM pg_authid;
```
按如下修改改为md5验证
```
vim /etc/postgresql/14/main/postgresql.conf 
listen_addresses = '*'  # 监听所有端口
password_encryption = md5  # 用于benchmark连接
```
### 1.2. 配置优化 单机，timescaledb （timescaledb-tune）
```
timescaledb-tune -conf-path /etc/postgresql/14/main/postgresql.conf --quiet --yes
```
### 1.3. 配置优化，分布式 timescaledb
```
max_prepared_transactions = 150  # data nodes
enable_partitionwise_aggregate = on  # access node
jit = off  # access node
statement_timeout = 0  # data nodes
wal_level=logical  # data nodes
default_transaction_isolation  # 没看懂咋修改...
```
# 基础命令
```
# 重启
systemctl restart postgresql
# 修改postgres用户密码
ALTER USER postgres WITH PASSWORD 'postgres';
# 创建数据库
CREATE database test;
# 创建timescaledb扩展
\c test;
CREATE EXTENSION IF NOT EXISTS timescaledb;
# 进入cli
su postgres -c 'psql -d test'
# 删除表
drop table test;
# 删除数据库
drop database test;
# cli命令
\l  # 列出数据库
\c  # 进入数据库
\dt  # 列出表
\dx  # 显示扩展
```

# 分布式操作
## 1. 添加数据节点
```
# SELECT add_data_node('node1', host => '172.20.31.67');
SELECT add_data_node('node1', host => '172.20.31.68');
SELECT add_data_node('node2', host => '172.20.31.69');
# 第二种写法（最简单）
SELECT add_data_node('node2', '172.20.31.69');
# 完整
SELECT add_data_node('node2','172.20.31.69','test',5432,	false,true,'postgres');

SELECT add_data_node('node1', '192.168.130.37');
SELECT add_data_node('node2', '192.168.130.38');
SELECT add_data_node('node3', '192.168.130.39');


SELECT add_data_node('node1','192.168.130.37','test',5432,false,true,'postgres');
SELECT add_data_node('node2','192.168.130.38','test',5432,false,true,'postgres');
SELECT add_data_node('node3','192.168.130.39','test',5432,false,true,'postgres');
```
## 1.1 查询数据节点
```
SELECT * FROM "timescaledb_information"."data_nodes";
```
## 1.2 删除节点
```
## 迁移数据 (测试失败)
CALL timescaledb_experimental.move_chunk('_timescaledb_internal._dist_hyper_1_1_chunk', 'node1', 'node2');
## 迁移数据失败之后的处理方式（测试失败）
CALL timescaledb_experimental.cleanup_copy_chunk_operation('ts_copy_1_31');
## 删除节点，local
SELECT delete_data_node('node2');
SELECT delete_data_node('node2', force => true);  # 强力删除
# 删除节点，集群
SELECT detach_data_node ( 'node1' , hypertable = > 'test' ); 
```
## 1.3 将数据节点给到超级表（可选）
```
# 创建表之后新增节点
SELECT attach_data_node('node3', hypertable => 'hypertable_name');
```
## 1.4 参考链接
https://www.bookstack.cn/read/TimescaleDB-2.1-en/spilt.3.55581822f2599198.md
## 2. 创建基础表
```
CREATE TABLE test (
 time        TIMESTAMPTZ       NOT NULL,
 location    TEXT              NOT NULL,
 temperature DOUBLE PRECISION  NULL
);
```

## 3. 创建一个跨多个数据节点扩展的分布式超表
```
SELECT create_distributed_hypertable('test', 'time', 'location');
SELECT create_distributed_hypertable('test', 'time', 'location', replication_factor => 3);  # 副本数
SELECT create_distributed_hypertable('test', 'time', 'location', data_nodes => '{ "node1", "node2", "node3" }', replication_factor => 3);  # 节点
```
## 3.1 创建超级表
```
SELECT create_hypertable('test', 'time', 'location',1);
```
## 3.2 给超级表添加列
```
ALTER TABLE test ADD COLUMN humidity DOUBLE PRECISION NULL;
```
## 3.3 删除超级表
同删除表

## 4 插入数据
```
INSERT INTO test VALUES ('2020-12-14 13:45', 1, '88');
INSERT INTO test VALUES ('2020-12-14 13:45', 2, '89');
INSERT INTO test VALUES ('2020-12-14 13:45', 3, '90');
```

## 5 查询数据
```
select * from test;
```

## 6.元数据操作
```
# 列出表的分布式信息
SELECT hypertable_name, data_nodes FROM timescaledb_information.hypertables WHERE hypertable_name = 'test';
# 列出表的chunk信息，chunk所在的节点
SELECT chunk_name, data_nodes FROM timescaledb_information.chunks WHERE hypertable_name = 'test';
# 查询基于时间的间隔长度(微秒)
SELECT h.table_name, c.interval_length FROM _timescaledb_catalog.dimension c JOIN _timescaledb_catalog.hypertable h ON h.id = c.hypertable_id;
```

# 分布式限制
https://docs.timescale.com/timescaledb/latest/overview/limitations/



