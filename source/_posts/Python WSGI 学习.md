---
updated: '2017-03-20 08:00:00'
categories: code
excerpt: WSGI 于 Python 就像 Servlet 于 Java，是介于 WebServer 与 Python 应用框架间的交互接口规范（Web Server Gateway Interface），也就是说 WebServer 和框架都需要对 WSGI 进行实现，好在 WSGI 的定义十分易于实现，所以目前主流的 WebServer 和 Python 都能对其提供支持。
date: '2017-03-20 08:00:00'
tags:
  - Python
urlname: python-wsgi
title: Python WSGI 学习
---

# 基本概念


[PEP 3333](https://www.python.org/dev/peps/pep-3333/)


WSGI 于 Python 就像 Servlet 于 Java，是介于 WebServer 与 Python 应用框架间的交互接口规范（Web Server Gateway Interface），也就是说 WebServer 和框架都需要对 WSGI 进行实现，好在 WSGI 的定义十分易于实现，所以目前主流的 WebServer 和 Python 都能对其提供支持。


# WSGI Server


符合 WSGI 规范的 WebServer 称为 WSGI Server，它的作用是接受客户端的 request 请求并将相关环境变量按照 WSGI 规范调用 WSGI Application，并将处理得到的 response 返回给客户端。


比如 `apache + mod_wsgi` 或者 Python 内置的 wsgiref。


wsgiref 是纯 Python 实现的 WSGI Server，它完全按照 WSGI 标准进行设计，但是未考虑任何运行效率，仅提供开发和测试使用。


# WSGI Application


运行于 WSGI Server 上的 Python 程序称为 WSGI Application，其最基本的形式是：


```python
# hello.py
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return '<h1>Hello world</h1>'
```


这个 application 函数需要由 WSGI Server 调用：


```python
# server.py
from wsgiref.simple_server import make_server
from hello import application

httpd = make_server('', 8000, application)
print "Serving HTTP on port 8000..."
httpd.serve_forever()
```


可以看出 WSGI Application 实际上是一个 callable 对象，比如一个实现了 **call** 的实例。


# Middleware


Middleware 指的是对 application 的包装，使得在不修改现有 application 实现的情况下，对其增加新的功能，比如


```python
# goodbye.py
class Goodbye(object):
    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        response = self.app(environ, start_response)
        return response.replace("Hello", "Goodbye")
```


```python
# server.py
from wsgiref.simple_server import make_server
from hello import application
from goodbye import Goodbye

app = Goodbye(application)
httpd = make_server('', 8000, app)
print "Serving HTTP on port 8000..."
httpd.serve_forever()
```


在此基础上可以进一步实现参数解析、路由分发、Cookies、Session、模板等，就是各个 Web 框架自己的实现了。


# uWSGI & uwsgi


uWSGI 是一个 WEB 服务器，它实现了 WSGI、http、uwsgi 等协议，用于接收前端的请求转发给后端 Python 框架。


一般 Web 应用中，最外层是 Nginx / Apache，中间是 uWSGI，最内是各种 Python 框架(Django, Flask).


uwsgi 是 uWSGI 的协议，用于 uWSGI 与前端 WebServer 的通信，

