# MongoDB

## Database

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

### Initialization, Startup, Login, Account

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

## Database and Collection

```shell
show databases; # 也可以用show dbs
db.dropDatabase() # drop a existing database.
# If a database does not exist, MongoDB creates the database when 
# you first store data for that database
> use test
switched to db test

db.createCollection(name, options) //Options parameter is optional
db.createCollection("mycol", 
{ capped : true, autoIndexID : true, size : 6142800, max : 10000 } )
{
"ok" : 0,
"errmsg" : "BSON field 'create.autoIndexID' is an unknown field.",
"code" : 40415,
"codeName" : "Location40415
"
}
# drop a collection
db.mycollection.drop()
```

# Create, Retrieve, Update, Delete

```shell
> db.person.insert({"name":"john", "age": 21})
WriteResult({ "nInserted" : 1 })
> db.person.insert({"name":"john", "birthplace": "UK"})
WriteResult({ "nInserted" : 1 })
> show collections;
person
> db.person.find();
{ "_id" : ObjectId("62ee8d47c493891f7bfdb4d2"), "name" : "john", "age" : 21 }
{ "_id" : ObjectId("62ee8d67c493891f7bfdb4d3"), "name" : "john", "birthplace" : "UK" }

> db.person.insertOne({"name": "ben"})
{
    "acknowledged" : true,
    "insertedId" : ObjectId("62ef94e25bad651083bba2a0")
}

# remove one record
> db.COLLECTION_NAME.remove(DELETION_CRITERIA,1)

# Remove All Documents
db.COLLECTION_NAME.remove({})

# find records with specific fields
db.COLLECTION_NAME.find({},{KEY:1})

db.mycol.findOne({title: "MongoDB Overview"})

# Following is the basic syntax of AND
db.mycol.find({ $and: [ {<key1>:<value1>}, { <key2>:<value2>} ] })

> db.COLLECTION_NAME.update(SELECTION_CRITERIA, UPDATED_DATA)

db.person.update({"_id": ObjectId("62ee8d67c493891f7bfdb4d3")}, {$set:{"name":"Mike"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

# By default, MongoDB will update only a single document. To update multiple documents, you need to set a parameter 'multi' to true.
>db.mycol.update({'title':'MongoDB Overview'},
   {$set:{'title':'New MongoDB Tutorial'}},{multi:true}
)

# find use date comparation
db.area.find({
    "establishedDate":
        {
            $gt: ISODate("1776-07-03T15:54:17Z")
        }
})

# update or create a new record
db.person.save({ "_id" : ObjectId("62ee8d67c493891f7bfdb4d3"), "name" : "Mike" })

> db.empDetails.updateOne(
    {First_Name: 'Radhika'},
    { $set: { Age: '30',e_mail: 'radhika_newemail@gmail.com'}}
)

> db.empDetails.updateMany(
    {Age:{ $gt: "25" }},
    { $set: { Age: '20'}}
)

# projection.
db.person.find({}, {"name": 1, "_id": 0})
{ "name" : "john" }
{ "name" : "Mike" }
{ "name" : "ben" }

# limit
> db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
db.mycol.find({},{"title":1,_id:0}).limit(1).skip(1)
{"title":"NoSQL Overview"}

# sort
> db.COLLECTION_NAME.find().sort({KEY:1})
> db.country.find({}, {"name": 1, "_id": 0, "population": 1}).sort({"name": 1, "population": 1})
{ "name" : "china", "population" : 1390000000 }
{ "name" : "global", "population" : NumberLong("10000000000") }
{ "name" : "global", "population" : NumberLong("11000000000") }

# update or insert. Atomic Operation
# insert if Canada doesn't exsist or increase population with 1 if it exists
db.country.findAndModify({
    "query":{ "name": "Canada"},
    "update": {"$inc" : {"population": 1}},
    "upsert": true
})


db.country.findAndModify({
    query: {name: "Canada"},
    update: {
        $set: {
            continent: "North America",
            name: "Canada",
            population: 18000000,
            establishedDate: ISODate("1867-06-30T16:00:00.186Z")
        }
    },
    upsert: true
})
```

### insert, insertOne 和 insertMany区别

 The insert() method is deprecated in major driver so you should use the the .insertOne() method whenever you want to insert a single document into your collection and the .insertMany when you want to insert multiple documents into your collection. Of course this is not mentioned in the documentation but the fact is that nobody really writes an application in the shell. The same thing applies to updateOne, updateMany, deleteOne, deleteMany, findOneAndDelete, findOneAndUpdate and findOneAndReplace.

[mongodb - What&#39;s the difference between insert(), insertOne(), and insertMany() method? - Stack Overflow](https://stackoverflow.com/questions/36792649/whats-the-difference-between-insert-insertone-and-insertmany-method)

## 索引

```shell
# create index, 1 is for ascending order, -1 is for descending order
> db.COLLECTION_NAME.createIndex({KEY:1})
> db.mycol.createIndex({"title":1,"description":-1})
> db.country.createIndex({"name": 1}) 
{
    "createdCollectionAutomatically" : false,
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 2,
    "ok" : 1
}


# drop index
> db.COLLECTION_NAME.dropIndex({KEY:1})

# Instead of the index specification document (above syntax), you can also specify the name of the index directly as:
dropIndex("name_of_the_index")
db.test.dropIndex({"name": 1})
{
    "ok" : 0,
    "errmsg" : "ns not found",
    "code" : 26,
    "codeName" : "NamespaceNotFound"
}

# drop multiple indexes
db.mycol.dropIndexes({"title":1,"description":-1})
{ "nIndexesWas" : 2, "ok" : 1 }

# get indexes
db.COLLECTION_NAME.getIndexes()
```

// todo 分析效果，删除索引的影响

## Aggregation

```shell
# basic syntax
db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)

# sum
db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}])

db.country.aggregate([{$group: {_id:"$name", population: {$sum:"$population"}}}])
{ "_id" : "global", "population" : NumberLong("21000000000") }
{ "_id" : "china", "population" : 1390000000 }


# 要返回所有人口超过1000万的州，使用下列聚合操作:
db.zipcodes.aggregate( { $group :
                         { _id : "$state",
                           totalPop : { $sum : "$pop" } } },
                       { $match : {totalPop : { $gte : 10*1000*1000 } } } )


# 要返回每个州的城市平均人口，请使用以下聚合操作:
db.zipcodes.aggregate( { $group :
                         { _id : { state : "$state", city : "$city" },
                           pop : { $sum : "$pop" } } },
                       { $group :
                       { _id : "$_id.state",
                         avgCityPop : { $avg : "$pop" } } } )

db.country.aggregate([{$group: {_id:{continent: "$continent", name :"$name"}, population: {$sum:"$population"}}}])
{ "_id" : { "continent" : "Asia", "name" : "India" }, "population" : NumberLong("10010000000") }
{ "_id" : { "continent" : "Asia", "name" : "china" }, "population" : 1390000000 }


# 统计数量
db.area.count({continent: "Asia"})



```

## Monitor

```shell
# This command checks the status of all running mongod instances and return counters of database operations.
mongostat -u learner  -p "study" --authenticationDatabase "admin"

# This command tracks and reports the read and write activity of MongoDB instance on a collection basis. 
mongotop -u learner  -p "study" --authenticationDatabase "admin"
```

## Java usage

```java
public class MongoApp {

  private static final Log log = LogFactory.getLog(MongoApp.class);

  public static void main(String[] args) throws Exception {

    MongoOperations mongoOps = new MongoTemplate(MongoClients.create(), "database");
    mongoOps.insert(new Person("Joe", 34));

    log.info(mongoOps.findOne(new Query(where("name").is("Joe")), Person.class));

    mongoOps.dropCollection("person");
  }
}
```

# 

# Reference

[MongoDB - Overview](https://www.tutorialspoint.com/mongodb/mongodb_overview.htm)

[MongoDB 用户名密码登录 - 简书](https://www.jianshu.com/p/79caa1cc49a5)

[无法连接到远程 mongodb 服务器](https://www.learnfk.com/question/mongodb/28002848.html)

[MongoDB配置用户名和密码进行认证登录 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1799755)

https://www.mongodb.com/docs/manual/reference/built-in-roles/

https://www.mongodb.com/docs/manual/reference/method/db.collection.findAndModify/

[聚合示例 — MongoDB Manual](https://mongodb-documentation.readthedocs.io/en/latest/tutorial/aggregation-examples.html#gsc.tab=0)

https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/

[让 MongoDB 的 CRUD 有 JPA 的味道 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1888798)
