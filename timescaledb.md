# pg配置修改
>. vim /etc/postgresql/14/main/pg_hba.conf 
host    all             all             0.0.0.0/0               trust  # 允许其他节点访问
vim /etc/postgresql/14/main/postgresql.conf 
listen_addresses = '0'  # 监听所有端口
# 配置优化 单机，timescaledb （timescaledb-tune）
timescaledb-tune -conf-path /etc/postgresql/14/main/postgresql.conf  --quiet
# 配置优化，分布式 timescaledb
max_prepared_transactions = 150  # data nodes
enable_partitionwise_aggregate = on  # access node
jit = off  # access node
statement_timeout = 0  # data nodes
wal_level=logical  # data nodes
default_transaction_isolation  # 没看懂咋修改...

# 重启
systemctl restart postgresql

# 创建数据库
CREATE database example;

# 创建timescaledb扩展
\c example
CREATE EXTENSION IF NOT EXISTS timescaledb;

# 进入cli
su postgres -c 'psql -d example'
## cli命令
\l   # 列出数据库
\c   # 进入数据库
\dt  # 列出表
\dx  # 显示扩展

# 创建基础表
CREATE TABLE example (
 time        TIMESTAMPTZ       NOT NULL,
 location    TEXT              NOT NULL,
 temperature DOUBLE PRECISION  NULL
);

# 添加数据节点
# SELECT add_data_node('node1', host => '172.20.31.67');
SELECT add_data_node('node1', host => '172.20.31.68');
SELECT add_data_node('node2', host => '172.20.31.69');
# 第二种写法
SELECT add_data_node('node2', '172.20.31.69');
# 完整
SELECT add_data_node('node2','172.20.31.69','example',5432,	false,true,'123456') 
## 删除节点



## 删除表
drop table example;
## 删除数据库
drop database example;

## 创建超级表
# SELECT create_hypertable('example', 'time', 'location',1);
# SELECT create_hypertable('example', 'time', 'location',2);
# SELECT create_hypertable('example', 'time', 'location',3);
# 创建一个跨多个数据节点扩展的分布式超表
SELECT create_distributed_hypertable('example', 'time', 'location');
SELECT create_distributed_hypertable('example', 'time', 'location', replication_factor => 3);  # 副本数
## 给超级表添加列
ALTER TABLE example ADD COLUMN humidity DOUBLE PRECISION NULL;
## 删除超级表，同删除表
## 查询基于时间的间隔长度(微秒)
SELECT h.table_name, c.interval_length FROM _timescaledb_catalog.dimension c JOIN _timescaledb_catalog.hypertable h ON h.id = c.hypertable_id;

插入数据
INSERT INTO example VALUES ('2020-12-14 13:45', 1, '88');
INSERT INTO example VALUES ('2020-12-14 13:45', 2, '89');
INSERT INTO example VALUES ('2020-12-14 13:45', 3, '90');

# 查询数据
select * from example;

# 列出表的分布式信息
SELECT hypertable_name, data_nodes FROM timescaledb_information.hypertables WHERE hypertable_name = 'example';
# 列出表的chunk信息
SELECT chunk_name, data_nodes FROM timescaledb_information.chunks WHERE hypertable_name = 'example';


# 添加数据节点
SELECT add_data_node('node3', host => '172.20.31.69');
# 将数据节点给到超级表
SELECT attach_data_node('node3', hypertable => 'hypertable_name');



# 删除数据节点
## 迁移数据 (没成功)
CALL timescaledb_experimental.move_chunk('_timescaledb_internal._dist_hyper_1_1_chunk', 'node1', 'node2');
## 
CALL timescaledb_experimental.cleanup_copy_chunk_operation('ts_copy_1_31');

## 删除节点
SELECT delete_data_node('node2');
SELECT delete_data_node('node2', force => true);  # 强力删除

SELECT detach_data_node ( 'node1' , hypertable = > 'example' ); 




