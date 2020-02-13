---
layout: post
title:  "微信公众号后台程序的Django版本实现"
date: 2019-02-20 11:15:00 +0800
categories: jekyll update
---
"微信公众号后台程序的Django版本实现"
===
1. 为何要写Django版本的微信后台程序
&emsp;&emsp;官方![](https://mp.weixin.qq.com/wiki)已经提供了详细的帮助文档和开发教程。关于微信后台如何工作，请仔细阅读官方帮助文档，本文档不会详细介绍。为什么还要写？当你阅读了开发者文档的入门指南，迫不及待的要用教程上面的例程运行时，发现上面的关键软件包web.py只支持python2（github上有pythhon3的版本，但试了没成功），而你的其它包很可能要用python3。首先python2已经不再推荐了，更糟糕的是当你想去找找有没有最新版本的web.py能支持python3的时候，发现web.py已经停止更新了（开发者已经狗带）。无奈之下心里默默的诅咒了一下腾讯，这个教程估计几年也不更新了。后来想是不是可以自己用Django框架写一个后台程序，这样可以使用Django的全部特性。本着不要重复造轮子的精神，将程序贴出来供有需要的程序猿参考，少走点弯路。

2. 手机和微信公众号交互信息的流程是什么？
&emsp;&emsp;当我们向微信公众号发送一条信息时，到底发生了什么？大致流程是手机发出的信息被腾讯的后台服务器收到，然后经过处理之后发到我们事先设置的服务器上；我们的服务器经过处理之后在发送回腾讯的服务器，腾讯服务器再将我们服务器发送的消息发回手机端。其中我们的服务器和腾讯的服务器中间走的是http协议（还必须是80端口，这点挺蛋疼的），这就是我们需要处理的部分，其余流程是腾讯自己的事情。因为中间是http协议，所以理论上任何web框架均可以作为开发程序。

3. 最小的Django程序
一般教程在写Django程序时需要有很多的配置文件，我们的目标只是写一个微信后台的Demo程序，所以不需要那么多的配置文件，可以用一个最小的Django程序开始，至于Django的相关介绍请自行学习。这里直接上代码：
```
# -*- coding: utf-8 -*-
# filename: main.py
#微信公众平台验证程序，参考微信mp平台文档从web.py修改为django
import sys
from handle import Handle
from django.conf import settings

settings.configure(
    DEBUG=True,
    ALLOWED_HOSTS=['118.89.144.1'],
    # SECURITY WARNING: keep the secret key used in production secret!
    SECRET_KEY = 's&!9ejh-hbr5op@z@2^_k!6uhy)ykm)gv=4+9&8c9gld0@q&ea',
    ROOT_URLCONF = __name__,
    MIDDLEWARE_CLASSES = (
        'django.middleware.common.CommonMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.middleware.clickjacking.XFrameOptionsMiddleware',
        )
    )   

from django.conf.urls import url
from django.http import HttpResponse

def index(request):
    MyHandle = Handle()
    return HttpResponse(MyHandle.GET(request))

urlpatterns = (
    url(r'^wx/$', index),
)

if __name__== '__main__':
    from django.core.management import execute_from_command_line
    execute_from_command_line(sys.argv)
```
以上的程序，只要再完成Handle()，就可以在命令行下执行python main.py runserver 0.0.0.0:80 启动。Handle程序见下面4。

4. 设置我们的服务器
&emsp;&emsp;实现后台微信公众号程序，我们必须有一台公网的服务器，而且程序还必须使用80端口（腾讯要求）。当你在进入腾讯设置页面设置自己的服务器时，会碰到token的概念。什么是token？token：只用于验证开发者服务器。也就是token不参与消息发送接收过程，在微信管理平台的“开发-基本配置”页面，可以设置开发者服务器的地址，但在修改时微信需要验证一下服务器（但我没看出来有什么必要），这时需要开发者服务器能响应微信管理平台的验证请求，token是微信和开发者服务器之间的一个约定。
&emsp;&emsp;所以，当你要设置服务器地址时，需要一个验证程序来证明这个服务器是你设置的，验证程序的详细说明参考官方文档的“入门指引”，这里直接列出Djiano版本的源代码以取代官方文档中“1.4开发者基本配置”的程序。
```    
# -*- coding: utf-8 -*-
# filename: handle.py
import hashlib
class Handle(object):
    def GET(self,request):
        try:
            #微信认证应该只支持GET方式
            if request.method == 'GET':
                #print('request.GET=%s'%request.META['QUERY_STRING'])
                signature = request.GET.get('signature')
                signature = signature.encode('utf-8') #默认是utf-8的字符，转换为bytes，下同
                #print("signature=%s"% signature)
                timestamp = request.GET.get('timestamp')
                timestamp = timestamp.encode('utf-8')
                #print("timestamp=%s"% timestamp)
                nonce = request.GET.get('nonce')
                nonce = nonce.encode('utf-8')
                #print("nonce=%s"% nonce)
                echostr = request.GET.get('echostr')
                token = "xxxx" #请按照公众平台官网\基本配置中信息填写
                token = token.encode('utf-8')
                mylist = [token, timestamp, nonce]
                mylist.sort()
                sha1 = hashlib.sha1()
                list(map(sha1.update, mylist)) #python3的map用法和python2不同，map返回object
                hashcode = sha1.hexdigest()
                hashcode = hashcode.encode('utf-8')
                #print("handle/GET func: hashcode=%s, signature=%s "%( hashcode, signature))
                if hashcode == signature:
                    return echostr
                else:
                    return "check error"
        except Exception as e:
            #print(e)
            return e
```
5. 最简单的微信公众号完整后台程序
&emsp;&emsp;有了以上的基础就可以写一个最简单的微信公众号完整后台程序，该程序在粉丝发送任意消息给公众号之后，回复“测试”两个字。程序共4个文件：main.py程序同上面3；handle.py改成下面的程序；reply.py和receive.py同教程里面“2.5 码代码”的程序，这里不再重复。
```
# -*- coding: utf-8 -*-
# filename: handle.py

import hashlib
import reply
import receive
from django.utils.encoding import smart_bytes

class Handle(object):
    def POST(self,request):
        try:
            #粉丝的信息用POST方式发送给自己服务器
            if request.method == 'POST':
                webData = smart_bytes(request.body) #此处是关键，获得http头信息，并转换为字符串（去unicode），参考自强学堂
                print("Handle Post webdata is %s"% webData)   #后台打日志
                recMsg = receive.parse_xml(webData)
                if isinstance(recMsg, receive.Msg) and recMsg.MsgType == 'text':
                    toUser = recMsg.FromUserName
                    fromUser = recMsg.ToUserName
                    content = "测试"
                    replyMsg = reply.TextMsg(toUser, fromUser, content)
                    return replyMsg.send()
                else:
                    print("暂且不处理")
                    return "success"
        except Exception as e:
            print(e)
            return e
```

6. 后续
&emsp;&emsp;如果能完成上面的测试，恭喜你已经掌握了微信公众号开发后台程序的精髓，微信还支持图片、声音等多媒体信息，你可以结合Django的强大功能自由发挥了。等等，还有什么access_token？哦，忘了告诉你，如果要调用微信平台接口，你必须还要处理access_token（access_token的官方解释：公众号的全局唯一接口调用凭据，公众号调用各接口时都需使用access_token。access_token的有效期目前为2个小时，需定时刷新，重复获取将导致上次获取的access_token失效。）。这个等以后有机会继续介绍吧，至少你今天已经有收获了。本文也作为微信公众平台开发文档的一个补充，建议读者还是仔细阅读微信的官方文档。





