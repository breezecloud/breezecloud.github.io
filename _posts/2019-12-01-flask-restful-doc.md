---
layout: post
title:  "flask-restful用户指南-快速入门（摘自官方文档）"
date: 2019-09-01 08:15:00 +0800
categories: jekyll update
---

什么是Flask-RESTful
===
&emsp;&emsp;是对Flask的扩展，它增加了对快速构建RESTAPI的支持。它是一个轻量级的抽象，可以与现有的ORM/库一起工作。Flask-RESTful鼓励最佳实践与最低限度的设置。如果你熟悉Flask，Flask-RESTful应该很容易使用。
官方文档位于（ https://flask-restful.readthedocs.io/en/latest/ ） 

安装
===
&emsp;&emsp;pip安装
```
pip install flask-restful需要Flask 0.10
```
&emsp;&emsp;flask-restful需要Flask 0.10以上，Python version 2.7, 3.4, 3.5, 3.6 or 3.7

快速入门
===
* 最小的API
```
from flask import Flask
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

class HelloWorld(Resource):
    def get(self):
        return {'hello': 'world'}

api.add_resource(HelloWorld, '/')

if __name__ == '__main__':
    app.run(debug=True)
```
保存为api.py之后运行：
```
$ python api.py
 * Running on http://127.0.0.1:5000/
 * Restarting with reloader
```
然后在命令行下执行：```$ curl http://127.0.0.1:5000/```来验证是否输出```{"hello": "world"}```

资源路由
===
&emsp;&emsp;flask-RESTful提供的主要构件是资源。资源构建在Flask可插拔视图(http://flask.pocoo.org/docs/views/) 之上，只需在资源上定义方法，就可以方便地访问多个HTTP方法。todo应用程序的基本CRUD资源(当然)如下所示：
```
from flask import Flask, request
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

todos = {}

class TodoSimple(Resource):
    def get(self, todo_id):
        return {todo_id: todos[todo_id]}

    def put(self, todo_id):
        todos[todo_id] = request.form['data']
        return {todo_id: todos[todo_id]}

api.add_resource(TodoSimple, '/<string:todo_id>')

if __name__ == '__main__':
    app.run(debug=True)
```
你可以这样试一试：
```
$ curl http://localhost:5000/todo1 -d "data=Remember the milk" -X PUT
{"todo1": "Remember the milk"}
$ curl http://localhost:5000/todo1
{"todo1": "Remember the milk"}
$ curl http://localhost:5000/todo2 -d "data=Change my brakepads" -X PUT
{"todo2": "Change my brakepads"}
$ curl http://localhost:5000/todo2
{"todo2": "Change my brakepads"}
```
如果安装了requests库，也可以从python执行：
```
>>> from requests import put, get
>>> put('http://localhost:5000/todo1', data={'data': 'Remember the milk'}).json()
{u'todo1': u'Remember the milk'}
>>> get('http://localhost:5000/todo1').json()
{u'todo1': u'Remember the milk'}
>>> put('http://localhost:5000/todo2', data={'data': 'Change my brakepads'}).json()
{u'todo2': u'Change my brakepads'}
>>> get('http://localhost:5000/todo2').json()
{u'todo2': u'Change my brakepads'}
```
flask-RESTful从视图方法中理解多种类型的返回值。类似于Flask，您可以返回任何可迭代的包括原始的flask响应对象,并且它将被转换为response。flask-RESTful还支持使用多个返回值设置响应代码和响应头，如下所示：
```
class Todo1(Resource):
    def get(self):
        # Default to 200 OK
        return {'task': 'Hello world'}

class Todo2(Resource):
    def get(self):
        # Set the response code to 201
        return {'task': 'Hello world'}, 201

class Todo3(Resource):
    def get(self):
        # Set the response code to 201 and return custom headers
        return {'task': 'Hello world'}, 201, {'Etag': 'some-opaque-string'}
```

Endpoints
===
&emsp;&emsp;很多时候，在API中，您的资源将有多个URL。可以将多个URL传递给Api对象上的add_resources()方法。每一个都会被路由到你的资源。
```
api.add_resource(HelloWorld,
    '/',
    '/hello')
```
可以将路径的部分作为变量匹配到资源方法中。
```
api.add_resource(Todo,
    '/todo/<int:todo_id>', endpoint='todo_ep')
```

参数语法
===
&emsp;&emsp;虽然Flask提供了对请求数据（即querystring或表单后编码数据）的简单访问，但验证表单数据仍然是一个难题。Flask RESTful内置了对使用类似于argparse的库进行请求数据验证的支持。
```
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('rate', type=int, help='Rate to charge for this resource')
args = parser.parse_args()
```
注意：
与argparse模块不同，reqparse.RequestParser.parse_args）返回一个Python字典，而不是自定义数据结构。

使用reqparse模块还可以免费提供正常的错误消息。如果参数未能通过验证，Flask RESTful将响应400个错误请求和一个突出显示错误的响应。
```
$ curl -d 'rate=foo' http://127.0.0.1:5000/todos
{'status': 400, 'message': 'foo cannot be converted to int'}
```
inputs模块提供了许多包含的公共转换函数，如inputs.date()和inputs.url()。

使用strict=True调用parse_args可确保在请求包含解析器未定义的参数时抛出错误。
```
args = parser.parse_args(strict=True)
```

数据格式
===
&emsp;&emsp;默认情况下，返回的可迭代中的所有字段都将按原样呈现。虽然这在处理Python数据结构时非常有效，但在处理对象时可能会变得非常令人沮丧。为了解决这个问题，Flask RESTful提供了fields模块和marshal_with（）装饰。与Django ORM和WTForm类似，您使用fields模块来描述你响应的结构。
```
from flask_restful import fields, marshal_with

resource_fields = {
    'task':   fields.String,
    'uri':    fields.Url('todo_ep')
}

class TodoDao(object):
    def __init__(self, todo_id, task):
        self.todo_id = todo_id
        self.task = task

        # This field will not be sent in the response
        self.status = 'active'

class Todo(Resource):
    @marshal_with(resource_fields)
    def get(self, **kwargs):
        return TodoDao(todo_id='my_todo', task='Remember the milk')
```        
上面的示例接受一个python对象并准备将其序列化。marshal_with（）装饰将应用于resource_fields字段描述的转换。从对象中提取的唯一字段是task。fields.Url字段是一个特殊的字段，它接受一个endpoint名称，并在响应中为该端点生成一个Url。您需要的许多字段类型已经包括在内。有关完整列表，请参阅“fields”指南。

完整示例
===
将此示例保存在api.py中
```    
from flask import Flask
from flask_restful import reqparse, abort, Api, Resource

app = Flask(__name__)
api = Api(app)

TODOS = {
    'todo1': {'task': 'build an API'},
    'todo2': {'task': '?????'},
    'todo3': {'task': 'profit!'},
}


def abort_if_todo_doesnt_exist(todo_id):
    if todo_id not in TODOS:
        abort(404, message="Todo {} doesn't exist".format(todo_id))

parser = reqparse.RequestParser()
parser.add_argument('task')


# Todo
# shows a single todo item and lets you delete a todo item
class Todo(Resource):
    def get(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        return TODOS[todo_id]

    def delete(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        del TODOS[todo_id]
        return '', 204

    def put(self, todo_id):
        args = parser.parse_args()
        task = {'task': args['task']}
        TODOS[todo_id] = task
        return task, 201


# TodoList
# shows a list of all todos, and lets you POST to add new tasks
class TodoList(Resource):
    def get(self):
        return TODOS

    def post(self):
        args = parser.parse_args()
        todo_id = int(max(TODOS.keys()).lstrip('todo')) + 1
        todo_id = 'todo%i' % todo_id
        TODOS[todo_id] = {'task': args['task']}
        return TODOS[todo_id], 201

##
## Actually setup the Api resource routing here
##
api.add_resource(TodoList, '/todos')
api.add_resource(Todo, '/todos/<todo_id>')


if __name__ == '__main__':
    app.run(debug=True)
```    

示例用法
```  
$ python api.py
 * Running on http://127.0.0.1:5000/
 * Restarting with reloader
```  
获取列表
```
$ curl http://localhost:5000/todos
{"todo1": {"task": "build an API"}, "todo3": {"task": "profit!"}, "todo2": {"task": "?????"}}
```
获取单个任务
```
$ curl http://localhost:5000/todos/todo3
{"task": "profit!"}
```
删除任务
```
$ curl http://localhost:5000/todos/todo2 -X DELETE -v

> DELETE /todos/todo2 HTTP/1.1
> User-Agent: curl/7.19.7 (universal-apple-darwin10.0) libcurl/7.19.7 OpenSSL/0.9.8l zlib/1.2.3
> Host: localhost:5000
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 204 NO CONTENT
< Content-Type: application/json
< Content-Length: 0
< Server: Werkzeug/0.8.3 Python/2.7.2
< Date: Mon, 01 Oct 2012 22:10:32 GMT
```
增加一个新任务
```
$ curl http://localhost:5000/todos -d "task=something new" -X POST -v

> POST /todos HTTP/1.1
> User-Agent: curl/7.19.7 (universal-apple-darwin10.0) libcurl/7.19.7 OpenSSL/0.9.8l zlib/1.2.3
> Host: localhost:5000
> Accept: */*
> Content-Length: 18
> Content-Type: application/x-www-form-urlencoded
>
* HTTP 1.0, assume close after body
< HTTP/1.0 201 CREATED
< Content-Type: application/json
< Content-Length: 25
< Server: Werkzeug/0.8.3 Python/2.7.2
< Date: Mon, 01 Oct 2012 22:12:58 GMT
<
* Closing connection #0
{"task": "something new"}
```
更新一个任务
```
$ curl http://localhost:5000/todos/todo3 -d "task=something different" -X PUT -v

> PUT /todos/todo3 HTTP/1.1
> Host: localhost:5000
> Accept: */*
> Content-Length: 20
> Content-Type: application/x-www-form-urlencoded
>
* HTTP 1.0, assume close after body
< HTTP/1.0 201 CREATED
< Content-Type: application/json
< Content-Length: 27
< Server: Werkzeug/0.8.3 Python/2.7.3
< Date: Mon, 01 Oct 2012 22:13:00 GMT
<
* Closing connection #0
{"task": "something different"}
```