## What is Elastic Search?

Elasticsearch is the distributed search and analytics engine at the heart of the Elastic Stack. Logstash and Beats facilitate collecting, aggregating, and enriching your data and storing it in Elasticsearch. Kibana enables you to interactively explore, visualize, and share insights into your data and manage and monitor the stack. Elasticsearch is where the indexing, search, and analysis magic happens.

Elasticsearch provides near real-time search and analytics for all types of data.



##  Data in: documents and indices

Elasticsearch is a distributed document store. Instead of storing information as rows of columnar data, Elasticsearch stores complex data structures that have been serialized as JSON documents.

Elasticsearch uses a data structure called an inverted index that supports very fast full-text searches.

An index can be thought of as an optimized collection of documents and each document is a collection of fields, which are the key-value pairs that contain your data. **By default, Elasticsearch indexes all data in every field** and each indexed field has a dedicated, optimized data structure.  For example, text fields are stored in inverted indices, and numeric and geo fields are stored in BKD trees. The ability to use the per-field data structures to assemble and return search results is what makes Elasticsearch so fast.

Elasticsearch also has the ability to be schema-less, which means that documents can be indexed without explicitly specifying how to handle each of the different fields that might occur in a document. 

前面看到往Elasticsearch里插入一条记录，其实就是直接PUT一个json的对象，这个对象有多个fields，比如上面例子中的*name, sex, age, about, interests*，那么在插入这些数据到Elasticsearch的同时，Elasticsearch还默默[1](https://link.jianshu.com?t=https://neway6655.github.io/elasticsearch/2015/09/11/elasticsearch-study-notes.html#fn:1)的为这些字段建立索引–倒排索引，因为Elasticsearch最核心功能是搜索。



## Information out: search and analyze

The Elasticsearch REST APIs support structured queries, full text queries, and complex queries that combine the two. Structured queries are similar to the types of queries you can construct in SQL. Full-text queries find all documents that match the query string and return them sorted by *relevance*—how good a match they are for your search terms.

In addition to searching for individual terms, you can perform phrase searches, similarity searches, and prefix searches, and get autocomplete suggestions.

## Scalability and resilience: clusters, nodes, and shards

Elasticsearch is built to be always available and to scale with your needs. It does this by being distributed by nature. You can add servers (nodes) to a cluster to increase capacity and Elasticsearch automatically distributes your data and query load across all of the available nodes.

An Elasticsearch index is really just a logical grouping of one or more physical shards, where each shard is actually a self-contained index. By distributing the documents in an index across multiple shards, and distributing those shards across multiple nodes, Elasticsearch can ensure redundancy, which both protects against hardware failures and increases query capacity as nodes are added to a cluster. As the cluster grows (or shrinks), Elasticsearch automatically migrates shards to rebalance the cluster.

There are two types of shards: primaries and replicas. Each document in an index belongs to one primary shard. A replica shard is a copy of a primary shard. Replicas provide redundant copies of your data to protect against hardware failure and increase capacity to serve read requests like searching or retrieving a document.

There are a number of performance considerations and trade offs with respect to shard size and the number of primary shards configured for an index. The more shards, the more overhead there is simply in maintaining those indices. The larger the shard size, the longer it takes to move shards around when Elasticsearch needs to rebalance a cluster.

### Cross-cluster replication (CCR)

CCR provides a way to automatically synchronize indices from your primary cluster to a secondary remote cluster that can serve as a hot backup. 

Cross-cluster replication is active-passive. The index on the primary cluster is the active leader index and handles all write requests. Indices replicated to secondary clusters are read-only followers.



## 命令

### Add a single document

```shell
curl -X POST "localhost:9200/logs-my_app-default/_doc?pretty" -H 'Content-Type: application/json' -d'
{
  "@timestamp": "2099-05-06T16:21:15.000Z",
  "event": {
    "original": "192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"
  }
}
'
```

the request automatically creates it using the built-in `logs-*-*` index template.

### Add multiple documents

Use the `_bulk` endpoint to add multiple documents in one request. Bulk data must be newline-delimited JSON (NDJSON). Each line must end in a newline character (`\n`), including the last line.

```shell
curl -X PUT "localhost:9200/logs-my_app-default/_bulk?pretty" -H 'Content-Type: application/json' -d'
{ "create": { } }
{ "@timestamp": "2099-05-07T16:24:32.000Z", "event": { "original": "192.0.2.242 - - [07/May/2020:16:24:32 -0500] \"GET /images/hm_nbg.jpg HTTP/1.0\" 304 0" } }
{ "create": { } }
{ "@timestamp": "2099-05-08T16:25:42.000Z", "event": { "original": "192.0.2.255 - - [08/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638" } }
'
```

###  Search data

Indexed documents are available for search in near real-time. The following search matches all log entries in `logs-my_app-default` and sorts them by `@timestamp` in descending order.

```shell
curl -X GET "localhost:9200/logs-my_app-default/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": { }
  },
  "sort": [
    {
      "@timestamp": "desc"
    }
  ]
}
'
```

####  Get specific fields

Parsing the entire `_source` is unwieldy for large documents. To exclude it from the response, set the `_source` parameter to `false`. Instead, use the `fields` parameter to retrieve the fields you want.

```shell
curl -X GET "localhost:9200/logs-my_app-default/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": { }
  },
  "fields": [
    "@timestamp"
  ],
  "_source": false,
  "sort": [
    {
      "@timestamp": "desc"
    }
  ]
}
'
```

####  Search a date range

To search across a specific time or IP range, use a `range` query.

```shell
curl -X GET "localhost:9200/logs-my_app-default/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "2099-05-05",
        "lt": "2099-05-08"
      }
    }
  },
  "fields": [
    "@timestamp"
  ],
  "_source": false,
  "sort": [
    {
      "@timestamp": "desc"
    }
  ]
}
'
```

## Query DSL (Domain Specific Language)

official doc: https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html

### term, match, match_phrase的区别

term是将传入的文本原封不动地（不分词）拿去查询。

match会对输入进行分词处理后再去查询，部分命中的结果也会按照评分由高到低显示出来。

match_phrase是按短语查询，只有存在这个短语的文档才会被显示出来。

也就是说，term和match_phrase都可以用于精确匹配，而match用于模糊匹配。

之前我以为match_phrase不会被分词，看来理解错了，其官方解释如下：

Like the match query, the match_phrase query first analyzes the query string to produce a list of terms. It then searches for all the terms, but keeps only documents that contain all of the search terms, in the same positions relative to each other.

总结下，这段话的3个要点：

1. match_phrase还是分词后去搜的
2. 目标文档需要包含分词后的所有词
3. 目标文档还要保持这些词的相对顺序和文档中的一致

只有当这三个条件满足，才会命中文档！



##  测试分析器

有些时候很难理解分词的过程和实际被存储到索引中的词条，特别是你刚接触Elasticsearch。为了理解发生了什么，你可以使用 `analyze` API 来看文本是如何被分析的。在消息体里，指定分析器和要分析的文本：

```sense
GET /_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}

GET ds-logs-my_app-default/_analyze
{
    "field": "team_name",
    "text":"Rocket"
}
```



## 参考文献

https://www.elastic.co/

[Elasticsearch是如何做到快速索引的](https://www.jianshu.com/p/ed7e1ebb2fb7)

https://github.com/xr2117/ElasticSearch7

[es中match_phrase和term区别_timothytt的博客-程序员宅基地](https://www.cxyzjd.com/article/timothytt/86775114)

https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html