# MongoDB

## ## Database

Database is a physical container for collections. Each database gets its own set of files on the file system. A single MongoDB server typically has multiple databases.

## Collection

Collection is a group of MongoDB documents. It is the equivalent of an RDBMS table. A collection exists within a single database. Collections do not enforce a schema. Documents within a collection can have different fields. Typically, all documents in a collection are of similar or related purpose.

## Document

A document is a set of key-value pairs. Documents have dynamic schema. Dynamic schema means that documents in the same collection do not need to have the same set of fields or structure, and common fields in a collection's documents may hold different types of data.

The following table shows the relationship of RDBMS terminology with MongoDB.

| RDBMS                      | MongoDB                                                  |
| -------------------------- | -------------------------------------------------------- |
| Database                   | Database                                                 |
| Table                      | Collection                                               |
| Tuple/Row                  | Document                                                 |
| column                     | Field                                                    |
| Table Join                 | Embedded Documents                                       |
| Primary Key                | Primary Key (Default key _id provided by MongoDB itself) |
| Database Server and Client |                                                          |
| mysqld/Oracle              | mongod                                                   |
| mysql/sqlplus              | mongo                                                    |

## Command

```shell
# 修改配置，否者内网其他机器无法访问
vim /etc/mongod.conf 
vim /etc/mongodb.conf
net:
  port: 27017
#  bindIp: 127.0.0.1
  bindIp: 0.0.0.0

# Start MongoDB
sudo service mongodb start

# Stop MongoDB
sudo service mongodb stop

# Restart MongoDB
sudo service mongodb restart

# To use MongoDB run the following command.
mongo
mongo --host $hostName --port $portNumber -u $username -p $password

# 创建admin账户，该账户可以反问admin库
use admin

db.createUser(
  {
    user: "adminUser",
    pwd: "adminPass",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)

# role可以设置成root

mongo --host localhost --port 27017 -u learner -p study  --authenticationDatabase 'admin'

# use某个database，然后认证
db.auth("username", "password")

# 查看状态

db.stats()
{
    "db" : "test",
    "collections" : 0,
    "views" : 0,
    "objects" : 0,
    "avgObjSize" : 0,
    "dataSize" : 0,
    "storageSize" : 0,
    "numExtents" : 0,
    "indexes" : 0,
    "indexSize" : 0,
    "fileSize" : 0,
    "fsUsedSize" : 0,
    "fsTotalSize" : 0,
    "ok" : 1
}
```

# Reference

[MongoDB - Overview](https://www.tutorialspoint.com/mongodb/mongodb_overview.htm)

[MongoDB 用户名密码登录 - 简书](https://www.jianshu.com/p/79caa1cc49a5)

[无法连接到远程 mongodb 服务器](https://www.learnfk.com/question/mongodb/28002848.html)

[MongoDB配置用户名和密码进行认证登录 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1799755)

https://www.mongodb.com/docs/manual/reference/built-in-roles/
