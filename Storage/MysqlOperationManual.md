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
mysql -u root -p password crm_pro < db.sql
```

##### 例子

```
mysqldump -h 192.168.2.240 -P3302 -u testvivopush -p  --no-data --add-drop-table=false ${database_name} > ${db.sql}
```

# 修改表结构

alter table tablename modify column id int not null primary key auto_increment

………………………..add id int not …………………………………………………

………………………..drop primary key

# 设置参数

set global max_connection=5000

# 查看状态

#### 查看数据库连接状态

select substring_index(host, ":", 1) ip, count(*) num from information_schema.processlist group by ip order by num desc

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
