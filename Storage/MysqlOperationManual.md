# 授权

#### 查看权限列表

select * from mysql.user;
select * from mysql.db;

#### 授权操作

#### 创建一个用户

CREATE USER '${username}'@'${host}' IDENTIFIED BY '${password}''

##### 例子

```
CREATE USER 'monty'@'localhost' IDENTIFIED BY 'some_pass';
CREATE USER 'monty'@'%' IDENTIFIED BY 'some_pass';
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'admin_pass';
CREATE USER 'dummy'@'localhost';
CREATE USER 'custom'@'localhost' IDENTIFIED BY 'obscure'
```

#### 修改一个用户密码

```
// 修改test用户密码为test
update mysql.user set password=password("test") where user="test";
```

#### 对某个用户授权

GRANT ${privileges} ON ${database}.${table} TO '${username}'@'${host}' [WITH GRANT OPTION]

##### 例子

```
GRANT ALL PRIVILEGES ON *.* TO 'monty'@'%' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'monty'@'10.%' WITH GRANT OPTION;
GRANT RELOAD,PROCESS ON *.* TO 'admin'@'localhost';
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP ON bankaccount.* to 'monty'@'10.%';
```

#### 刷新权限

flush privileges;

# Ubuntu开放端口

```shell
#防火墙开放3306端口
iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
#修改MySQL监听
vi /etc/mysql/mysql.conf.d/mysqld.cnf
##注释掉bind-address = 127.0.0.1
#重启动MySQL
sudo /etc/init.d/mysql restart
#开放MySQL用户远程访问权限
grant all privileges on *.* to 'root'@'%' identified by '123456';
flush privileges;
```

# 导出导入数据

#### 导出

```
mysqldump [-u... -p...] db_name [tbl_name ...] [> mydb_tables.sql]
mysqldump [options] --databases db_name ...
mysqldump [options] --all-databases
```

##### 导出csv

```
select * from vivopush_pz.client_info into outfile '/tmp/client_info.csv' fields terminated by '\,';
```

#### 导入

```
# bash命令
mysql -u root -p password crm_pro < db.sql

# 登录mysql，选择具体数据库后，执行外部sql
source /home/test/user-data.sql
```

##### 例子

```
mysqldump -h 192.168.2.240 -P3302 -u testvivopush -p  --no-data --add-drop-table=false ${database_name} > ${db.sql}
```

# 修改表结构

alter table tablename modify column id int not null primary key auto_increment

………………………..add id int not …………………………………………………

………………………..drop primary key

## 新增索引最佳实践
- MySQL 5.6 以上大部分alter操作，不需要加锁，默认算法是并行索引创建，在执行新增索引操作时，在 ALTER TABLE 语句中添加 ALGORITHM=INPLACE, LOCK=NONE, ONLINE=ON 选项，指定使用 INPLACE 算法、禁用锁定、使用在线模式等选项，让 MySQL 引擎使用并行操作方式来创建索引，从而减少锁定表的时间。

- 从 MySQL 8.0.12 开始，引入了 instant 算法并且默认使用。目前 instant 算法只支持增加列等少量 DDL 类型的操作，其他类型仍然会默认使用 inplace。

ALTER TABLE tbl_name DROP INDEX i1, ADD INDEX i1(key_part,...)
USING BTREE, ALGORITHM=INPLACE, LOCK=NONE;
• ALGORITHM可选: INPLACE / COPY
• LOCK可选: NONE SHARED 等加锁情况 -> 在 ALTER TABLE 语句上指定一个子句，如 LOCK = NONE (许可读和写)或 LOCK = SHARED (许可读)。如果请求的并发级别不可用，操作将立即停止。

ALGORITHM=INPLACE
更优秀的解决方案，在当前表加索引，步骤：
1.创建索引(二级索引)数据字典
1. 加共享表锁，禁止DML，允许查询
2. 读取聚簇索引，构造新的索引项，排序并插
入新索引
1. 等待打开当前表的所有只读事务提交
2. 创建索引结束

ALGORITHM=COPY
通过临时表创建索引，需要多一倍存储，还有更多的IO，步骤：
1. 新建带索引（主键索引）的临时表
2. 锁原表，禁止DML，允许查询
3. 将原表数据拷贝到临时表
4. 禁止读写,进行rename，升级字典锁
5. 完成创建索引操作
 
LOCK=DEFAULT：默认方式，MySQL自行判断使用哪种LOCK模式，尽量不锁表
LOCK=NONE：无锁：允许Online DDL期间进行并发读写操作。如果Online DDL操
作不支持对表的继续写入，则DDL操作失败，对表修改无效
LOCK=SHARED：共享锁：Online DDL操作期间堵塞写入，不影响读取
LOCK=EXCLUSIVE：排它锁：Online DDL操作期间不允许对锁表进行任何操作

### 参考
https://developer.aliyun.com/article/1111994

https://cloud.tencent.com/developer/article/1801049

https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-operations.html

https://stackoverflow.com/questions/35424543/alter-table-without-locking-the-entire-table

## 索引修改

```sql
CREATE INDEX indexName ON table_name (column_name)

ALTER table tableName ADD INDEX indexName(columnName)

DROP INDEX [indexName] ON mytable; 

CREATE TABLE mytable(  
  ID INT NOT NULL,   
  username VARCHAR(16) NOT NULL,  
  INDEX [indexName] (username(length)),
  UNIQUE [indexName] (username(length)));  
  CREATE UNIQUE INDEX indexName ON mytable(username(length)
) 

ALTER table mytable ADD UNIQUE [indexName] (username(length))

ALTER TABLE `tbl_feeds`ADD INDEX `IX_Feeds_username` (`username`) ,ADD INDEX `IX_Feeds_userid` (`userid`) ,ADD INDEX `IX_Feeds_content` (`content`);
```

# 设置参数
```bash
# 查看最大连接数
show variables like '%max_connection%';

# 在/etc/my.cnf里面设置数据库的最大连接数
[mysqld]
max_connections = 1000

# 或则mysql命令行
set global max_connections=5000
```
# 查看状态

#### 查看数据库连接状态
```bash

show global status like 'Thread%';
select substring_index(host, ":", 1) ip, count(*) num from information_schema.processlist group by ip order by num desc

```
# Query 诊断分析工具

```shell
// 查看是否开启profiling
select @@profiling;
// 开启 profiling
set profiling=1;
// execute sql
show profiles
// 查询第二条语句的执行情况
show profile for query 2;
// 指定资源类型查看
show profile cpu,swaps for query 2;
```
