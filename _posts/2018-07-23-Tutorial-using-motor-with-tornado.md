---
layout: post
title:  "Tutorial: Using Motor With Tornado教程：在Tornado中使用Motor(英汉对照)"
date: 2018-07-23 20:25:06 -0700
---

Contents
====
Tutorial: Using Motor With Tornado
* Tutorial Prerequisites
* Object Hierarchy
* Creating a Client
* Getting a Database
* Tornado Application Startup Sequence
* Getting a Collection
* Inserting a Document
* Getting a Single Document With find_one()
* Querying for More Than One Document
   * async for
   * Iteration in Python 3.4
* Counting Documents
* Updating Documents
* Removing Documents
* Commands
* Further Reading

Tutorial Prerequisites
===
准备
You can learn about MongoDB with the MongoDB Tutorial before you learn Motor.
Install pip and then do:
安装pip并安装tornado和motor：
```
$ pip install tornado motor
```
Once done, the following should run in the Python shell without raising an exception:
接着可以执行如下命令：
```
>>> import motor.motor_tornado
```
This tutorial also assumes that a MongoDB instance is running on the default host and port. Assuming you have downloaded and installed MongoDB, you can start it like so:
本教程假设MongoDB已经在本机缺省端口运行。你可以安装MongoDB并启动。
```
$ mongod
```

Object Hierarchy
====
对象层级：  
Motor, like PyMongo, represents data with a 4-level object hierarchy:  
* MotorClient represents a mongod process, or a cluster of them. You explicitly create one of these client objects, connect it to a running mongod or mongods, and use it for the lifetime of your application.
* MotorDatabase: Each mongod has a set of databases (distinct sets of data files on disk). You can get a reference to a database from a client.
* MotorCollection: A database has a set of collections, which contain documents; you get a reference to a collection from a database.
* MotorCursor: Executing find() on a MotorCollection gets a MotorCursor, which represents the set of documents matching a query.
* MotorClient 代表mongod进程，或者是它们的集群。您显式创建这些客户端对象中的一个，将其连接到运行的mongod，并将其用于应用程序的生命周期。
* MotorDatabase：每个mongod都有一组数据库（磁盘上的不同数据文件集）。您可以从客户端获得对数据库的引用。
* MotorCollection：数据库有一组集合，其中包含文档；从数据库中获取对集合的引用。
* MotorCursor：在一个MotorCollection 上执行find()，得到一个游标，它代表一组匹配查询的文档。

Creating a Client
====
建立一个客户端  
You typically create a single instance of MotorClient at the time your application starts up.  
在应用程序启动时，通常会创建一个MotorClient 实例。  
```
>>> client = motor.motor_tornado.MotorClient()
```
This connects to a mongod listening on the default host and port. You can specify the host and port like: 
这连接到一个mongod 监听默认主机和端口。您可以指定主机和端口类似：
```
>>> client = motor.motor_tornado.MotorClient('localhost', 27017)
```
Motor also supports connection URIs:
还支持连接URI：
```
>>> client = motor.motor_tornado.MotorClient('mongodb://localhost:27017')
```
Connect to a replica set like: 
连接到复制集：
```
>>> client = motor.motor_tornado.MotorClient('mongodb://host1,host2/?replicaSet=my-replicaset-name')
```

Getting a Database
====
创建数据库引用
A single instance of MongoDB can support multiple independent databases. From an open client, you can get a reference to a particular database with dot-notation or bracket-notation:
MongoDB的一个实例可以支持多个独立的数据库。在一个已经打开的客户端，您可以使用点标记或括号符号来获得对特定数据库的引用一个特定数据库，：
```
>>> db = client.test_database
>>> db = client['test_database']
```
Creating a reference to a database does no I/O and does not require an await expression.
创建数据库引用不需要I/O，也不需要await 表达式。

Tornado Application Startup Sequence
====
Now that we can create a client and get a database, we're ready to start a Tornado application that uses Motor:
现在我们可以创建一个客户端并获得一个数据库，我们准备启动一个使用Motor的Tornado应用程序：
```
db = motor.motor_tornado.MotorClient().test_database

application = tornado.web.Application([
    (r'/', MainHandler)
], db=db)

application.listen(8888)
tornado.ioloop.IOLoop.current().start()
```
There are two things to note in this code. First, the MotorClient constructor doesn't actually connect to the server; the client will initiate a connection when you attempt the first operation. Second, passing the database as the dbkeyword argument to Application makes it available to request handlers:
在这段代码中有两件事需要注意。首先，MotorClient 构造函数实际上没有连接到服务器；当您尝试第一次操作时，客户端将启动连接。第二，将数据库作为db参数传递给应用程序，使得它可以用于请求处理程序：
```
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        db = self.settings['db']
```
It is a common mistake to create a new client object for every request; this comes at a dire performance cost. Create the client when your application starts and reuse that one client for the lifetime of the process, as shown in these examples.
The Tornado HTTPServer class's start() method is a simple way to fork multiple web servers and use all of your machine's CPUs. However, you must create your MotorClient after forking:
为每个请求创建新的客户端对象是一个常见错误；这是一个可怕的性能代价。在应用程序启动时创建客户端，并在进程的生命周期中重用一个客户端，正如这些示例所示。
Tornado 中HTTPSServer类的start（）方法是一个简单的复制Web服务器进程方法来并发使用CPU的，您必须在复制后创建您的MotorClient ：
```
# Create the application before creating a MotorClient.
application = tornado.web.Application([
    (r'/', MainHandler)
])

server = tornado.httpserver.HTTPServer(application)
server.bind(8888)

# Forks one process per CPU.
server.start(0)

# Now, in each child process, create a MotorClient.
application.settings['db'] = MotorClient().test_database
IOLoop.current().start()
```
For production-ready, multiple-CPU deployments of Tornado there are better methods than HTTPServer.start(). See Tornado's guide to Running and deploying.
对于生产环境、多CPU部署的Tornado，有比HTTPSServer更好的方法。参考Tornado的运行和部署指南。

Getting a Collection
====
获得Collection
A collection is a group of documents stored in MongoDB, and can be thought of as roughly the equivalent of a table in a relational database. Getting a collection in Motor works the same as getting a database:
collection 是存储在MongoDB中的一组文档，并且可以被认为大致等同于关系数据库中的表。在Motor中获得一个collection与获取数据库方法类似：
```
>>> collection = db.test_collection
>>> collection = db['test_collection']
```
Just like getting a reference to a database, getting a reference to a collection does no I/O and doesn't require an await expression.
和获取数据库引用类似，创建collection引用不需要I/O，也不需要await 表达式。

Inserting a Document
====
插入文档（Document）
As in PyMongo, Motor represents MongoDB documents with Python dictionaries. To store a document in MongoDB, call insert_one() in an await expression:
和PyMongo一样，Motor用Python字典表示文档。若要在MongoDB中存储文档，请在await 表达式中调用insert_one()。
```
>>> async def do_insert():
...     document = {'key': 'value'}
...     result = await db.test_collection.insert_one(document)
...     print('result %s' % repr(result.inserted_id))
...
>>>
>>> IOLoop.current().run_sync(do_insert)
result ObjectId('...')
```
See also
 
The MongoDB documentation on
 参考MongoDB文档
insert
A typical beginner's mistake with Motor is to insert documents in a loop, not waiting for each insert to complete before beginning the next:
一个典型的初学者的错误是在一个循环中插入文档，而不是等待每个插入在下一个开始之前完成：
```
>>> for i in range(2000):
...     db.test_collection.insert_one({'i': i})
```
In PyMongo this would insert each document in turn using a single socket, but Motor attempts to run all the insert_one() operations at once. This requires up to max_pool_size open sockets connected to MongoDB, which taxes the client and server. To ensure instead that all inserts run in sequence, use await:
在PyMongo中，这将使用单个套接字依次插入每个文档，但Motor试图同时运行所有的insert_one()操作。这样会使连接到MongoDB的开放套接字到max_pool_size而耗尽客户端和服务器端的连接。为了确保所有插入都按顺序运行，请使用await：
```
>>> async def do_insert():
...     for i in range(2000):
...         await db.test_collection.insert_one({'i': i})
...
>>> IOLoop.current().run_sync(do_insert)
```
See also
参考：
Bulk Write Operations.
See also
The MongoDB documentation on
参考：
insert  
For better performance, insert documents in large batches with insert_many():
为了获得更好的性能，使用insert_many()插入大批量的文档：
```
>>> async def do_insert():
...     result = await db.test_collection.insert_many(
...         [{'i': i} for i in range(2000)])
...     print('inserted %d docs' % (len(result.inserted_ids),))
...
>>> IOLoop.current().run_sync(do_insert)
inserted 2000 docs
```

Getting a Single Document With find_one()
====
使用find_one()查询一个文档
Use find_one() to get the first document that matches a query. For example, to get a document where the value for key "i" is less than 1:
使用find_one()获取与查询匹配的第一个文档。例如，获取一个文档，其中关键字"i"的值小于1：
```
>>> async def do_find_one():
...     document = await db.test_collection.find_one({'i': {'$lt': 1}})
...     pprint.pprint(document)
...
>>> IOLoop.current().run_sync(do_find_one)
{'_id': ObjectId('...'), 'i': 0}
```
The result is a dictionary matching the one that we inserted previously.
The returned document contains an "_id", which was automatically added on insert.
(We use pprint here instead of print to ensure the document's key names are sorted the same in your output as ours.)
结果是一个字典与我们先前插入的字典相匹配。
返回的文档包含一个"_id"，它被自动添加到INSERT中。
（我们在这里使用pprint 代替打印，以确保文档的键名在输出中与我们的相同。）
See also
 
The MongoDB documentation on
 参考：
find

Querying for More Than One Document
====
多文档查询
Use find() to query for a set of documents. find() does no I/O and does not require an await expression. It merely creates an MotorCursor instance. The query is actually executed on the server when you call to_list() or execute an async for loop.
To find all documents with "i" less than 5:
使用find() 查询一组文档。find() 没有I/O，不需要await 表达式。它只创建一个MotorCursor 游标实例。当您调用l to_list()或为循环执行异步时(async for loop)，查询实际上在服务器上执行。
查询"i"小于5所有文档：
```
>>> async def do_find():
...     cursor = db.test_collection.find({'i': {'$lt': 5}}).sort('i')
...     for document in await cursor.to_list(length=100):
...         pprint.pprint(document)
...
>>> IOLoop.current().run_sync(do_find)
{'_id': ObjectId('...'), 'i': 0}
{'_id': ObjectId('...'), 'i': 1}
{'_id': ObjectId('...'), 'i': 2}
{'_id': ObjectId('...'), 'i': 3}
{'_id': ObjectId('...'), 'i': 4}
```
A length argument is required when you call to_list to prevent Motor from buffering an unlimited number of documents.
length 参数可以在调用to_list 返回无限数量的文档时保护Motor缓冲。
* async for  
You can handle one document at a time in an async for loop:
您可以在async for循环中处理一个文档 ：
```
>>> async def do_find():
...     c = db.test_collection
...     async for document in c.find({'i': {'$lt': 2}}):
...         pprint.pprint(document)
...
>>> IOLoop.current().run_sync(do_find)
{'_id': ObjectId('...'), 'i': 0}
{'_id': ObjectId('...'), 'i': 1}
```
You can apply a sort, limit, or skip to a query before you begin iterating:
在开始迭代之前，可以对查询应用排序、限制或跳过：
```
>>> async def do_find():
...     cursor = db.test_collection.find({'i': {'$lt': 4}})
...     # Modify the query before iterating
...     cursor.sort('i', -1).skip(1).limit(2)
...     async for document in cursor:
...         pprint.pprint(document)
...
>>> IOLoop.current().run_sync(do_find)
{'_id': ObjectId('...'), 'i': 2}
{'_id': ObjectId('...'), 'i': 1}
```
The cursor does not actually retrieve each document from the server individually; it gets documents efficiently in large batches.
游标实际上没有从服务器中单独检索每个文档；它在批量查询中有效地获取文档。
* Iteration in Python 3.4  
In Python versions without async for, handle one document at a time with fetch_next and next_object():
在没有ASYNC的Python版本中，一次处理一个文档，使用fetch_next和next_object()：
```
>>> @gen.coroutine
... def do_find():
...     cursor = db.test_collection.find({'i': {'$lt': 5}})
...     while (yield cursor.fetch_next):
...         document = cursor.next_object()
...         pprint.pprint(document)
...
>>> IOLoop.current().run_sync(do_find)
{'_id': ObjectId('...'), 'i': 0}
{'_id': ObjectId('...'), 'i': 1}
{'_id': ObjectId('...'), 'i': 2}
{'_id': ObjectId('...'), 'i': 3}
{'_id': ObjectId('...'), 'i': 4}
```
Counting Documents
====
文档计数
Use count_documents() to determine the number of documents in a collection, or the number of documents that match a query:
使用count_documents()来确定集合中的文档数量，或者确定与查询匹配的文档数量：
```
>>> async def do_count():
...     n = await db.test_collection.count_documents({})
...     print('%s documents in collection' % n)
...     n = await db.test_collection.count_documents({'i': {'$gt': 1000}})
...     print('%s documents where i > 1000' % n)
...
>>> IOLoop.current().run_sync(do_count)
2000 documents in collection
999 documents where i > 1000
```
Updating Documents
====
更改文件
replace_one() changes a document. It requires two parameters: a query that specifies which document to replace, and a replacement document. The query follows the same syntax as for find() or find_one(). To replace a document:
replace_one()更改文档。它需要两个参数：一个指定要替换哪个文档的查询，以及一个替换文档。查询遵循与 find()或 find_one()相同的语法。替换一个文档：
```
>>> async def do_replace():
...     coll = db.test_collection
...     old_document = await coll.find_one({'i': 50})
...     print('found document: %s' % pprint.pformat(old_document))
...     _id = old_document['_id']
...     result = await coll.replace_one({'_id': _id}, {'key': 'value'})
...     print('replaced %s document' % result.modified_count)
...     new_document = await coll.find_one({'_id': _id})
...     print('document is now %s' % pprint.pformat(new_document))
...
>>> IOLoop.current().run_sync(do_replace)
found document: {'_id': ObjectId('...'), 'i': 50}
replaced 1 document
document is now {'_id': ObjectId('...'), 'key': 'value'}
```
You can see that replace_one() replaced everything in the old document except its _id with the new document.
Use update_one() with MongoDB's modifier operators to update part of a document and leave the rest intact. We'll find the document whose "i" is 51 and use the $set operator to set "key" to "value":
可以看到，replace_one()替换了旧文档中的所有内容，除了它的的_ID。
使用update_one()使用MongoDB的修饰操作符来更新文档的一部分，并将其余部分保留完整。我们将找到其"i"为51的文档，并使用$set操作符将"key"设置为"value"：
```
>>> async def do_update():
...     coll = db.test_collection
...     result = await coll.update_one({'i': 51}, {'$set': {'key': 'value'}})
...     print('updated %s document' % result.modified_count)
...     new_document = await coll.find_one({'i': 51})
...     print('document is now %s' % pprint.pformat(new_document))
...
>>> IOLoop.current().run_sync(do_update)
updated 1 document
document is now {'_id': ObjectId('...'), 'i': 51, 'key': 'value'}
"key" is set to "value" and "i" is still 51.
update_one() only affects the first document it finds, you can update all of them with update_many():
```
"key"被设置为"value"，"i"仍然是51。
update_one()只影响它找到的第一个文档，可以用update_many()更新所有的文档：
```
await coll.update_many({'i': {'$gt': 100}},
                       {'$set': {'key': 'value'}})
```
See also
 
The MongoDB documentation on
 参考
update

Removing Documents
====
删除文档
delete_many() takes a query with the same syntax as find(). delete_many() immediately removes all matching documents.
delete_many()使用与find()相同的语法进行查询。delete_many()（）立即删除所有匹配的文档。
```
>>> async def do_delete_many():
...     coll = db.test_collection
...     n = await coll.count_documents({})
...     print('%s documents before calling delete_many()' % n)
...     result = await db.test_collection.delete_many({'i': {'$gte': 1000}})
...     print('%s documents after' % (await coll.count_documents({})))
...
>>> IOLoop.current().run_sync(do_delete_many)
2000 documents before calling delete_many()
1000 documents after
```
See also
 参考
The MongoDB documentation on
 
remove

Commands
====
命令
All operations on MongoDB are implemented internally as commands. Run them using the command() method onMotorDatabase:
MongoDB上的所有操作都作为命令内部实现。使用MotorDatabase的 command()方法运行它们：
```
.. doctest:: after-inserting-2000-docs
>>> from bson import SON
>>> async def use_distinct_command():
...     response = await db.command(SON([("distinct", "test_collection"),
...                                      ("key", "i")]))
...
>>> IOLoop.current().run_sync(use_distinct_command)
```
Since the order of command parameters matters, don't use a Python dict to pass the command's parameters. Instead, make a habit of using bson.SON, from the bson module included with PyMongo.
Many commands have special helper methods, such as create_collection() or aggregate(), but these are just conveniences atop the basic command() method.
由于命令参数的顺序很重要，所以不要使用Python dict来传递命令的参数。取而代之的是，养成使用包含在PyMongo的bson 模块的bson.SON习惯，。
许多命令都有特殊的帮助方法，如 create_collection()或aggregate()，但这些方便的命令基于基本 command()方法。
See also
 
The MongoDB documentation on
 参考：
commands

Further Reading
====
The handful of classes and methods introduced here are sufficient for daily tasks. The API documentation for MotorClient, MotorDatabase, MotorCollection, and MotorCursor provides a reference to Motor's complete feature set.
Learning to use the MongoDB driver is just the beginning, of course. For in-depth instruction in MongoDB itself, see The MongoDB Manual.
这里介绍的少数类和方法对于日常任务来说是足够的。MotorClient、MotorDatabase、MotorCollection、MotorCursor 等的API文档为Motor的完整特征集提供了参考。
当然，学习使用MongoDB驱动程序仅仅是个开始。对于MongoDB本身的深入研究，请参见MongoDB手册

&emsp;&emsp;本人的更多原创文章请加入个人微信公众号。  
![](/images/weixin.jpg)
