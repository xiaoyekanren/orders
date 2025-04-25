# PostgreSQL

## 简单命令
``` shell
# 进入postgresql
psql -U 用户 -d 数据库

# 列出数据库
\l

# 使用数据库
\c zzm

# 列出库中的表
\dt

# 导出表
pg_dump -U postgres -d zzm -t shsw1701 -f /var/lib/postgresql/results/shsw1701;
## U 用户
## d 数据库
## t 表名
## f 导出路径

# 创建表
CREATE TABLE shsw1701(
	time	timestamp NOT NULL,
	ZT8	int4 NOT NULL,
	XN502	float4 NOT NULL,
	XN511	int4 NOT NULL
);

# 新增列
ALTER TABLE shsw1715 ADD SubwayNum varchar;
# 编辑列, 给shsw1713表的subway列，统一修改为'shsw1713'
UPDATE shsw1713 SET SubwayNum= 'shsw1713';

# 导出一个库
pg_dump -U username -d databasename -f ./databasename
# 导出一张表
pg_dump -U username -d databasename -t aaa -f ./aaa.sql
# 导出整库
pg_dumpall -f aaaa
# 导入表, 将shsw17文件导入到postgres的某张表中(无需建表)
psql -U postgres -d shsw < ./shsw17 

# 初始化数据库
initdb --username=postgres --pwprompt ../data/
--username 指定超级用户为postgres
--pwprompt 指定超级用户密码
../data/ 指定数据路径为这个

#  删除数据，快速清理, delete from table_name之后，空间不会马上释放掉，这时需要输入下面语句
VACUUM FULL TABLE_NAME

# 将查询结果写入另一张表, 这个地方前后字段要保持一致，不能是*
insert INTO shsw17(subwaynum,time,zt8) select subwaynum,time,zt8 from shsw1701;
```
