# MySQL

## 修改root密码
``` shell
use mysql;
update user set password=password("123456") where user="root";
flush privileges;
```

## root允许远程访问
``` shell
use mysql;
update user set host='%' where user='root';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
flush privileges;
# 这地方可以查询一下mysql数据库的user表
select User,Host from mysql.user;
# 若有空白用户，删除
# 若有root，localhost，确认下权限是不是all
```

## 导出数据+结构
``` shell
# 单表
mysqldump -uroot -p123456 flok_app >flok_app
# 多表
mysqldump -uroot -p123456 --databases flok_app flok_master hive test_lishuang test_zongzan tianyuan > test.sql
# 全库备份
mysqldump -uroot -p --all-databases ./all_db.sql
# 导入数据
mysql -uroot -p flok_app < flok_app
```

## Linux-忘记Mysql密码
``` shell
# 1. 停止mysql服务
# 2. 跳过验证登陆
cd $MYSQL_HOME/sbin
./mysqld --skip-grant-tables
# 3. 跳过验证登陆
cd $MYSQL_HOME/bin
./mysql -uroot
# 4. 更新root密码
use mysql;
update user set password=password("new_pass") where user="root";
flush privileges;
```

## 设置全局变量
当前server生效，重启server失效。  
``` shell
# 查询timeout相关的全局变量：
show variables like '%timeout%';

# set
set global connect_timeout= 30;
set global net_read_timeout = 60;
set global net_write_timeout = 120;
``` 

若重启生效需要修改配置文件，修改完后重启server。  
``` shell
# 若重启生效需要修改配置文件，在/etc/mysql.cnf或任意incloude的文件里写入
[mysqld]
connect_timeout = 30
net_read_timeout = 60
net_write_timeout = 120
```
