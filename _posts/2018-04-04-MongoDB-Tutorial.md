---
title: MongoDB之瞎搞篇
author: Kuang
categorie: Database
tag: MongoDB
---

我们在做的农业知识图谱的文档有点多了，之前都是直接存txt，没有存在数据库里，随着后来这些文档越来越多，还是搞个数据库统一管理一下。在数据库选择方面，虽然用传统的关系型数据库也没啥问题，毕竟我们数据量不会特别多，但本着瞎折腾的精神，还是试试文档数据库mongodb吧(似乎最近这个数据库争议比较大，有要凉的趋势)。



### 分析

当然，我们不仅仅是为了套用一下nosql装个X才选择的MongoDB，选择MongoDB的的理由首先是因为它性能优越，查询速度快。还有一个重要的原因是MongoDB作为Nosql的代表作，没有表结构，这意味着它很容易扩展。我们项目中的文件格式往往不统一，而且经常会有新的需求，如果采用Mysql可能就需要建立多个数据库或者频繁地修改表结构，很是不方便。因此我们最终决定上MongoDB。

### 安装

安装倒是很简单，直接去[MongoDB dowload center](https://www.mongodb.com/download-center)下载即可，下载解压，进入到bin目录运行```./mongod``` ,或者自定义号配置文件后运行```./mongod --config configFile```即可，然后运行MongoDB的shell```./mongo``` 就可以操纵数据库。但是我碰到了一个很蛋疼的问题，我第一次接触MongoDB的时候是大三，我记得那时候我运行MongoDB后，在浏览器中输入127.0.0.1:27017（或者是127.0.0.1:28017?)就可以有个可视化工具，但是这次运行报了sslhandshake failed 错误，应该是配置的问题，以前版本默认的配置就能直接用这个东西，但MongoDB3.6好像并不行，试着修改了一下配置文件没有成功，想了想还是直接用robo3t来可视化吧，这个坑先留着，日后有空再来解决。

为了避免每次启动都要进入bin目录下进行一系列操作，可以修改~/.bashrc，达到在任意目录下直接运行MongoDB的目的

```shell
vim ~/.bashrc
# 在末尾加上下面3句话
export MONGODB_HOME="/home/kuangjun/mongodb-linux-x86_64-ubuntu1604-3.6.3"
alias mongod=$MONGODB_HOME/bin/mongod
alias mongod=$MONGODB_HOME/bin/mongo
#　保存后退出后
source ~/.bashrc
# 之后只要在终端输入mongod，效果就等同与进入bin目录下执行./mongod; mongo同理
```



以下内容是看官方tutorial顺手做的笔记，水平有限，难免出错，欢迎指出。

－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－

###   MongoDB Overview

MongoDB is a cross-platform, document oriented database that provides, high performance, high availability, and easy scalability. MongoDB works on concept of **collection** and **document**.

下面是MongoDB的几个概念

#### Database

database是collections的物理容器，一个MongoDB server通常有多个database。

#### Collection

collection是一组MongoDB的documents，相当于RDBMS里面的一个表。但是collection并不强制使用统一的schema(与关系型数据库不同)，一个collection中的documents可以有不同的fields。但通常来说，一个collection里面的documents都有相似或相关的用途。

#### Document

document是一组key-value对，documents有动态的schema，动态的schema意味着同一个collection中的documents并不需要有相同的field或者结构，并且同一collection下，允许不同文档的相同的field中出现不同的数据类型。

|    RDBMS    |                 MongoDB                  |
| :---------: | :--------------------------------------: |
|  Database   |                 Database                 |
|    Table    |                Collection                |
|  Tuple/Row  |                 Document                 |
|   column    |                  Field                   |
| Table Join  |            Embedded Documents            |
| Primary Key | Primary Key (Default key _id provided by mongodb itself) |



#### Document举例

下面是一个document的样例

```json
{
   _id: ObjectId(7df78ad8902c)
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by: 'tutorials point',
   url: 'http://www.tutorialspoint.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100, 
   comments: [	
      {
         user:'user1',
         message: 'My first comment',
         dateCreated: new Date(2011,1,20,2,15),
         like: 0 
      },
      {
         user:'user2',
         message: 'My second comments',
         dateCreated: new Date(2011,1,25,7,45),
         like: 5
      }
   ]
}
```

其中\_id是一个12字节的十六进制数，是document的唯一标识。\_id可以自己定义也可以让mongodb自己生成。其中，前4个字节是时间戳，接下来的3个字节是machine id,接下来的2个字节是MongoDB server的进程id，剩下的3个字节是一个自增的数值。

### MongoDB数据模型

MongoDB中的数据具有灵活的schema。一个collection中的documents允许有不同的field和structure。不同documents的相同field可能有不同的数据类型。

### 创建数据库

创建

```
use DATABASE_NAME
```

查看目前使用的数据库

```
db
```

查看数据库列表

```
show dbs
```

至少插入一个document到数据库中，该数据库才算被创建，否则在show dbs时无法看到刚创建的数据库。

如果没有创建任何数据库，collection会存储在test database中。

### 删除数据库

```shell
#进入到要删除的database下，执行
db.dropDatabase()
```

### 创建collection

```shell
db.createCollection(name,options)
```

| Parameter | Type     | Description                              |
| --------- | -------- | ---------------------------------------- |
| Name      | String   | Name of the collection to be created     |
| Options   | Document | (Optional) Specify options about memory size and indexing |

参数options是可选的，也就是说你可以只定义name参数，使用默认的options参数，除非你又特殊的需要。

| Field       | Type    | Description                              |
| ----------- | ------- | ---------------------------------------- |
| capped      | Boolean | (Optional) If true, enables a capped collection. Capped collection is a fixed size collection that automatically overwrites its oldest entries when it reaches its maximum size. **If you specify true, you need to specify size parameter also.** |
| autoIndexId | Boolean | (Optional) If true, automatically create index on _id field.s Default value is false. |
| size        | number  | (Optional) Specifies a maximum size in bytes for a capped collection. **If capped is true, then you need to specify this field also.** |
| max         | number  | (Optional) Specifies the maximum number of documents allowed in the capped collection. |

查看collections列表

```shell
show collections
```

### 删除collection

```
db.COLLECTION_NAME.drop()
```

### 数据类型

- **String** − This is the most commonly used datatype to store the data. String in MongoDB must be UTF-8 valid.
- **Integer** − This type is used to store a numerical value. Integer can be 32 bit or 64 bit depending upon your server.
- **Boolean** − This type is used to store a boolean (true/ false) value.
- **Double** − This type is used to store floating point values.
- **Min/ Max keys** − This type is used to compare a value against the lowest and highest BSON elements.
- **Arrays** − This type is used to store arrays or list or multiple values into one key.
- **Timestamp** − ctimestamp. This can be handy for recording when a document has been modified or added.
- **Object** − This datatype is used for embedded documents.
- **Null** − This type is used to store a Null value.
- **Symbol** − This datatype is used identically to a string; however, it's generally reserved for languages that use a specific symbol type.
- **Date **− This datatype is used to store the current date or time in UNIX time format. You can specify your own date time by creating object of Date and passing day, month, year into it.
- **Object ID** − This datatype is used to store the document’s ID.
- **Binary data** − This datatype is used to store binary data.
- **Code** − This datatype is used to store JavaScript code into the document.
- **Regular expression** − This datatype is used to store regular expression.

### 插入document

```shell
db.COLLECTION_NAME.insert(document)
```

例子:插入一个document

```shell
db.mycol.insert({
   _id: ObjectId(7df78ad8902c),
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by: 'tutorials point',
   url: 'http://www.tutorialspoint.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
})
```

例子:插入多个document

```shell
db.post.insert([
   {
      title: 'MongoDB Overview', 
      description: 'MongoDB is no sql database',
      by: 'tutorials point',
      url: 'http://www.tutorialspoint.com',
      tags: ['mongodb', 'database', 'NoSQL'],
      likes: 100
   },
	
   {
      title: 'NoSQL Database', 
      description: "NoSQL database doesn't have tables",
      by: 'tutorials point',
      url: 'http://www.tutorialspoint.com',
      tags: ['mongodb', 'database', 'NoSQL'],
      likes: 20, 
      comments: [	
         {
            user:'user1',
            message: 'My first comment',
            dateCreated: new Date(2013,11,10,2,35),
            like: 0 
         }
      ]
   }
])
```

To insert the document you can use **db.post.save(document)** also. If you don't specify **_id** in the document then **save()** method will work same as **insert()** method. If you specify _id then it will replace whole data of document containing _id as specified in save() method.

### Query Document

```shell
db.COLLECTION_NAME.find()
```

find()函数查找出来的结果是无结构的，如果要使其有结构，使用

```shell
db.COLLECTION_NAME.find().pretty()
```

findOne()返回一条document

#### 查询条件

To query the document on the basis of some condition, you can use following operations.

| Operation           | Syntax                     | Example                                  | RDBMS Equivalent             |
| ------------------- | -------------------------- | ---------------------------------------- | ---------------------------- |
| Equality            | {\<key\>:\<value\>}        | db.mycol.find({"by":"tutorials point"}).pretty() | where by = 'tutorials point' |
| Less Than           | {\<key\>:{$lt:\<value\>}}  | db.mycol.find({"likes":{$lt:50}}).pretty() | where likes < 50             |
| Less Than Equals    | {\<key\>:{$lte:\<value\>}} | db.mycol.find({"likes":{$lte:50}}).pretty() | where likes <= 50            |
| Greater Than        | {\<key\>:{$gt:\<value\>}}  | db.mycol.find({"likes":{$gt:50}}).pretty() | where likes > 50             |
| Greater Than Equals | {\<key\>:{$gte:\<value\>}} | db.mycol.find({"likes":{$gte:50}}).pretty() | where likes >= 50            |
| Not Equals          | {\<key\>:{$ne:\<value\>}}  | db.mycol.find({"likes":{$ne:50}}).pretty() | where likes != 50            |

#### AND in MongoDB 

如果在使用查询条件时用","隔开多个条件，默认就是"AND"，当然也可以指定AND

```
db.COLLECTION_NAME.find(
   {
      $and: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

#### OR in MongoDB

```shell
db.mycol.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

#### 同时使用AND和OR

```shell
db.mycol.find({"likes": {$gt:10}, $or: [{"by": "tutorials point"},
   {"title": "MongoDB Overview"}]}).pretty()

```

### Update Document

```shell
db.COLLECTION_NAME.update(SELECTION_CRITERIA, UPDATED_DATA)
```

例子

假设mycol里面有如下数据

```shell
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"MongoDB Overview"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Tutorials Point Overview"}
```

将"MongoDB Overview"改成"New MongoDB Tutorial"

```shell
>db.mycol.update({'title':'MongoDB Overview'},{$set:{'title':'New MongoDB Tutorial'}})
>db.mycol.find()
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"New MongoDB Tutorial"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Tutorials Point Overview"}

```

update默认只更新一条document,如果想要更新多条document,则需要设置multi属性为true

```shell
db.mycol.update({'title':'MongoDB Overview'},
   {$set:{'title':'New MongoDB Tutorial'}},{multi:true})
```

#### MongoDB Save() Method

 **save()** 方法将已有的document替换成新的document

```shell
db.COLLECTION_NAME.save({_id:ObjectId(),NEW_DATA})
```

例子

```shell
>db.mycol.save(
   {
      "_id" : ObjectId(5983548781331adf45ec7), "title":"Tutorials Point New Topic",
         "by":"Tutorials Point"
   }
)
>db.mycol.find()
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"Tutorials Point New Topic",
   "by":"Tutorials Point"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Tutorials Point Overview"}
```



### MongoDB 删除文档

remove方法有两个参数:

* deletion criteria 符合条件的documents会全部删除
* justOne 如果设为true或者1的话，只会删除一个文档

例子:

假设现在mycol这个collection有以下数据

```shell
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"MongoDB Overview"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Tutorials Point Overview"}
```

现在要删除title是"MongoDB Overview"

```shell
>db.mycol.remove({'title':'MongoDB Overview'})
>db.mycol.find()
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Tutorials Point Overview"}
```

#### 只删除

#### 一个document

使用justOne参数删除所有满足条件的documents中的第一条

```shell
>db.COLLECTION_NAME.remove(DELETION_CRITERIA,1)
```

#### 删除所有的document

```shell
>db.mycol.remove()
>db.mycol.find()
>
```

### Projection

即选择document的部分fields

#### 使用find()方法

find()函数的第二个参数，第二个参数指定想要接受的fields。在MongoDB中，当你执行find()方法，可以指定每个field是0还是1，0表示隐藏这个field，1表示展示这个参数。

```~shell
db.COLLECTION_NAME.find({},{KEY:1})
```

例子

假设有如下数据

```shell
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"MongoDB Overview"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Tutorials Point Overview"}
```

执行find()语句得到以下结果

```shell
>db.mycol.find({},{"title":1,_id:0})
{"title":"MongoDB Overview"}
{"title":"NoSQL Overview"}
{"title":"Tutorials Point Overview"}
>
```

### Limit Records

使用limit方法限制查找的条数

```shell
>db.COLLECTION_NAME.find().limit(NUMBER)
```

例子，假设有如下数据

```shell
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"MongoDB Overview"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Tutorials Point Overview"}
```

只展示两条结果

```shell
>db.mycol.find({},{"title":1,_id:0}).limit(2)
{"title":"MongoDB Overview"}
{"title":"NoSQL Overview"}
>
```

#### MongoDB的Skip方法

```shell
> db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
```

下面的例子，只展示第二条document

```shell
>db.mycol.find({},{"title":1,_id:0}).limit(1).skip(1)
{"title":"NoSQL Overview"}
>
```

### Sort Record

使用sort 方法,1表示升序,-1表示降序

```shell
>db.COLLECTION_NAME.find().sort({KEY:1})
```

例子，有如下数据

```shell
{ "_id" : ObjectId(5983548781331adf45ec5), "title":"MongoDB Overview"}
{ "_id" : ObjectId(5983548781331adf45ec6), "title":"NoSQL Overview"}
{ "_id" : ObjectId(5983548781331adf45ec7), "title":"Tutorials Point Overview"}
```

```shell
>db.mycol.find({},{"title":1,_id:0}).sort({"title":-1})
{"title":"Tutorials Point Overview"}
{"title":"NoSQL Overview"}
{"title":"MongoDB Overview"}
>
```



### 索引

使用ensureIndex()方法来建立索引

```shell
>db.COLLECTION_NAME.ensureIndex({KEY:1})
```

其中表示升序,-1表示降序

例子

```shell
>db.mycol.ensureIndex({"title":1})
>
```

使用ensureIndex()方法，能传递多个fields，在多个fields上建立索引

```shell
>db.mycol.ensureIndex({"title":1,"description":-1})
>
```

ensureIndex()方法接受如下参数

| Parameter          | Type          | Description                              |
| ------------------ | ------------- | ---------------------------------------- |
| background         | Boolean       | Builds the index in the background so that building an index does not block other database activities. Specify true to build in the background. The default value is **false**. |
| unique             | Boolean       | Creates a unique index so that the collection will not accept insertion of documents where the index key or keys match an existing value in the index. Specify true to create a unique index. The default value is **false**. |
| name               | string        | The name of the index. If unspecified, MongoDB generates an index name by concatenating the names of the indexed fields and the sort order. |
| dropDups           | Boolean       | Creates a unique index on a field that may have duplicates. MongoDB indexes only the first occurrence of a key and removes all documents from the collection that contain subsequent occurrences of that key. Specify true to create unique index. The default value is **false**. |
| sparse             | Boolean       | If true, the index only references documents with the specified field. These indexes use less space but behave differently in some situations (particularly sorts). The default value is **false**. |
| expireAfterSeconds | integer       | Specifies a value, in seconds, as a TTL to control how long MongoDB retains documents in this collection. |
| v                  | index version | The index version number. The default index version depends on the version of MongoDB running when creating the index. |
| weights            | document      | The weight is a number ranging from 1 to 99,999 and denotes the significance of the field relative to the other indexed fields in terms of the score. |
| default_language   | string        | For a text index, the language that determines the list of stop words and the rules for the stemmer and tokenizer. The default value is **english**. |
| language_override  | string        | For a text index, specify the name of the field in the document that contains, the language to override the default language. The default value is language. |



### 聚集

聚集操作可以统计数据的一些信息，例如求和、求最大值、最小值等。

```shell
>db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```

例子，假设有如下数据

```shell
{
   _id: ObjectId(7df78ad8902c)
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by_user: 'tutorials point',
   url: 'http://www.tutorialspoint.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
},
{
   _id: ObjectId(7df78ad8902d)
   title: 'NoSQL Overview', 
   description: 'No sql database is very fast',
   by_user: 'tutorials point',
   url: 'http://www.tutorialspoint.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 10
},
{
   _id: ObjectId(7df78ad8902e)
   title: 'Neo4j Overview', 
   description: 'Neo4j is no sql database',
   by_user: 'Neo4j',
   url: 'http://www.neo4j.com',
   tags: ['neo4j', 'database', 'NoSQL'],
   likes: 750
},
```

如果想统计以上collection中，每个user写了多少tutorials,则可以使用以下聚集操作

```shell
> db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{
   "result" : [
      {
         "_id" : "tutorials point",
         "num_tutorial" : 2
      },
      {
         "_id" : "Neo4j",
         "num_tutorial" : 1
      }
   ],
   "ok" : 1
}
>
```

在SQL中，能达到同样效果的语句是 **select by_user, count(\*) from mycol group by by_user**

下面是所有的aggregation操作

| Expression | Description                              | Example                                  |
| ---------- | ---------------------------------------- | ---------------------------------------- |
| $sum       | Sums up the defined value from all documents in the collection. | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}]) |
| $avg       | Calculates the average of all given values from all documents in the collection. | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}]) |
| $min       | Gets the minimum of the corresponding values from all documents in the collection. | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}]) |
| $max       | Gets the maximum of the corresponding values from all documents in the collection. | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}]) |
| $push      | Inserts the value to an array in the resulting document. | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}]) |
| $addToSet  | Inserts the value to an array in the resulting document but does not create duplicates. | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}]) |
| $first     | Gets the first document from the source documents according to the grouping. Typically this makes only sense together with some previously applied “$sort”-stage. | db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}]) |
| $last      | Gets the last document from the source documents according to the grouping. Typically this makes only sense together with some previously applied “$sort”-stage. | db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}]) |

#### Pipeline概念

在shell中，pipeline指的是将一个指令的输出当做另一个指令输入。MongoDB同样在aggregation里支持这一操作。也就是说，某些aggregation操作将一组documents当做输入，并产生另一组documents

下面是所有支持pipeline的stages:

```shell
$project − Used to select some specific fields from a collection.

$match − This is a filtering operation and thus this can reduce the amount of documents that are given as input to the next stage.

$group − This does the actual aggregation as discussed above.

$sort − Sorts the documents.

$skip − With this, it is possible to skip forward in the list of documents for a given amount of documents.

$limit − This limits the amount of documents to look at, by the given number starting from the current positions.

$unwind − This is used to unwind document that are using arrays. When using an array, the data is kind of pre-joined and this operation will be undone with this to have individual documents again. Thus with this stage we will increase the amount of documents for the next stage.

```

### Replication

保证数据高可用和一致性用的，了解一下

设置Replica Set

* 首先，关闭正在运行的mongodb server
* 通过指定 -- replSet 选项来设置replica set

```shell
mongod --port "PORT" --dbpath "YOUR_DB_DATA_PATH" --replSet "REPLICA_SET_INSTANCE_NAME"
```

例子

```shell
mongod --port 27017 --dbpath "D:\set up\mongodb\data" --replSet rs0
```

#### Add Members to Replica Set

To add members to replica set, start mongod instances on multiple machines. Now start a mongo client and issue a command **rs.add()**.

##### Syntax

The basic syntax of **rs.add()** command is as follows −

```
>rs.add(HOST_NAME:PORT)
```

##### Example

Suppose your mongod instance name is **mongod1.net** and it is running on port **27017**. To add this instance to replica set, issue the command **rs.add()**in Mongo client.

```
>rs.add("mongod1.net:27017")
>
```

You can add mongod instance to replica set only when you are connected to primary node. To check whether you are connected to primary or not, issue the command **db.isMaster()** in mongo client.



后面的以后有空再写了,mmp
