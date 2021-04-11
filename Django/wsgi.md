---
author: "李昌"
title: "wsgi"
date: "2021-04-09"
tags: ["wsgi", "django", "web"]
categories: ["Django"]
ShowToc: true
TocOpen: true
---

> 转载自：https://segmentfault.com/a/1190000011365430

## 1. WSGI介绍
### 1.1 什么是WSGI
首先介绍几个关于WSGI相关的概念
WSGI：全称是`Web Server Gateway Interface`，`WSGI`不是服务器，`python`
模块，框架，API或者任何软件，只是一种规范，描述`web server`如何与`web application`通信的规范。`server`和`application`的规范在`PEP 3333`中有具体描述。要实现`WSGI`协议，必须同时实现`web server`和`web application`，当前运行在`WSGI`协议之上的web框架有`Torando,Flask,Django`

`uwsgi`:与`WSGI`一样是一种通信协议，是`uWSGI`服务器的独占协议，用于定义传输信息的类型(type of information)，每一个`uwsgi packet`前4byte为传输信息类型的描述，与`WSGI`协议是两种东西，据说该协议是`fcgi`协议的10倍快。

`uWSGI`：是一个web服务器，实现了`WSGI`协议、`uwsgi`协议、`http`协议等。

WSGI协议主要包括`server`和`application`两部分：
> `WSGI server`负责从客户端接收请求，将`request`转发给`application`，将`application`返回的`response`返回给客户端；
`WSGI application`接收由`server`转发的`request`，处理请求，并将处理结果返回给`server`。`application`中可以包括多个栈式的中间件(middlewares)，这些中间件需要同时实现`server`与`application`，因此可以在`WSGI`服务器与`WSGI`应用之间起调节作用：对服务器来说，中间件扮演应用程序，对应用程序来说，中间件扮演服务器。

`WSGI`协议其实是定义了一种`server`与`application`解耦的规范，即可以有多个实现`WSGI server`的服务器，也可以有多个实现`WSGI application`的框架，那么就可以选择任意的`server`和`applicatiodn`组合实现自己的web应用。例如`uWSGI`和`Gunicorn`都是实现了`WSGI server`协议的服务器，`Django`，`Flask`是实现了`WSGI application`协议的web框架，可以根据项目实际情况搭配使用。

以上介绍了相关的常识，接下来我们来看看如何简单实现WSGI协议。

### 1.2 怎么实现WSGI
上文说过，实现`WSGI`协议必须要有`wsgi server`和`application`，因此，我们就来实现这两个东西。  
我们来看看官方WSGI使用WSGI的wsgiref模块实现的小demo
```python
def demo_app(environ,start_response):  
    from StringIO import StringIO  
    stdout = StringIO()  
    print >>stdout, "Hello world!"  
    print >>stdout  
    h = environ.items(); h.sort()  
    for k,v in h:  
        print >>stdout, k,'=', repr(v)  
    start_response("200 OK", [('Content-Type','text/plain')])  
    return [stdout.getvalue()]  
  
httpd = make_server('localhost', 8002,  demo_app)  
httpd.serve_forever()  # 使用select  
```
实现了一个application，来获取客户端的环境和回调函数两个参数，以及httpd服务端的实现，我们来看看make_server的源代码
```python
def make_server(  
    host, port, app, server_class=WSGIServer, handler_class=WSGIRequestHandler):  
    """Create a new WSGI server listening on `host` and `port` for `app`"""  
    server = server_class((host, port), handler_class)  
    server.set_app(app)  
    return server
```

下面我们自己来实现一遍：
WSGI 规定每个 python 程序（Application）必须是一个可调用的对象（实现了__call__ 函数的方法或者类），接受两个参数 environ（WSGI 的环境信息） 和 start_response（开始响应请求的函数），并且返回 iterable。几点说明：
```
environ 和 start_response 由 http server 提供并实现
environ 变量是包含了环境信息的字典
Application 内部在返回前调用 start_response
start_response也是一个 callable，接受两个必须的参数，status（HTTP状态）和 response_headers（响应消息的头）
可调用对象要返回一个值，这个值是可迭代的。
```

> application

```python
 # 1. 可调用对象是一个函数
def application(environ, start_response):
 
    response_body = 'The request method was %s' % environ['REQUEST_METHOD']
    
    # HTTP response code and message
    status = '200 OK'
    
    # 应答的头部是一个列表，每对键值都必须是一个 tuple。
    response_headers = [('Content-Type', 'text/plain'),
                        ('Content-Length', str(len(response_body)))]
    
    # 调用服务器程序提供的 start_response，填入两个参数
    start_response(status, response_headers)
    
    # 返回必须是 iterable
    return [response_body]    
   
# 2. 可调用对象是一个类
class AppClass:
    """这里的可调用对象就是 AppClass 这个类，调用它就能生成可以迭代的结果。
        使用方法类似于： 
        for result in AppClass(env, start_response):
            do_somthing(result)
    """

    def __init__(self, environ, start_response):
        self.environ = environ
        self.start = start_response

    def __iter__(self):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield "Hello world!\n"
 
# 3. 可调用对象是一个实例 
class AppClass:
    """这里的可调用对象就是 AppClass 的实例，使用方法类似于： 
        app = AppClass()
        for result in app(environ, start_response):
             do_somthing(result)
    """
 
    def __init__(self):
        pass
 
    def __call__(self, environ, start_response):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield "Hello world!\n"
```

> server

上面已经说过，标准要能够确切地实行，必须要求程序端和服务器端共同遵守。上面提到， envrion 和 start_response 都是服务器端提供的。下面就看看，服务器端要履行的义务。
```
准备 environ 参数
定义 start_response 函数
调用程序端的可调用对象
```

```python
import os, sys
 
def run_with_cgi(application):    # application 是程序端的可调用对象
    # 准备 environ 参数，这是一个字典，里面的内容是一次 HTTP 请求的环境变量
    environ = dict(os.environ.items())
    environ['wsgi.input']        = sys.stdin
    environ['wsgi.errors']       = sys.stderr
    environ['wsgi.version']      = (1, 0)
    environ['wsgi.multithread']  = False
    environ['wsgi.multiprocess'] = True
    environ['wsgi.run_once']     = True            
    environ['wsgi.url_scheme'] = 'http'
 
    headers_set = []
    headers_sent = []
 
    # 把应答的结果输出到终端
    def write(data):
        sys.stdout.write(data)
        sys.stdout.flush()
 
    # 实现 start_response 函数，根据程序端传过来的 status 和 response_headers 参数，
    # 设置状态和头部
    def start_response(status, response_headers, exc_info=None):
        headers_set[:] = [status, response_headers]
          return write
 
    # 调用客户端的可调用对象，把准备好的参数传递过去
    result = application(environ, start_response)
    
    # 处理得到的结果，这里简单地把结果输出到标准输出。
    try:
        for data in result:
            if data:    # don't send headers until body appears
                write(data)
    finally:
        if hasattr(result, 'close'):
            result.close()
```

## 2. 由Django框架分析WSGI
下面我们以django为例，分析一下wsgi的整个流程

### 2.1 django WSGI application
`WSGI application`应该实现为一个可调用`iter`对象，例如函数、方法、类(包含**call**方法)。需要接收两个参数：一个字典，该字典可以包含了客户端请求的信息以及其他信息，可以认为是请求上下文，一般叫做`environment`（编码中多简写为`environ、env`），一个用于发送HTTP响应状态（`HTTP status`）、响应头（`HTTP headers`）的回调函数,也就是start_response()。通过回调函数将响应状态和响应头返回给server，同时返回响应正文(`response body`)，响应正文是可迭代的、并包含了多个字符串。
下面是`Django`中`application`的具体实现部分：
```python
# 继承, 但只实现了 __call__ 方法, 方便使用
class WSGIHandler(base.BaseHandler):
    initLock = Lock()
 
    # 可以将其视为一个代表 http 请求的类
    request_class = WSGIRequest
 
    # WSGIHandler 也可以作为函数来调用
    def __call__(self, environ, start_response):
        # Set up middleware if needed. We couldn't do this earlier, because
        # settings weren't available.
 
        # 这里的检测: 因为 self._request_middleware 是最后才设定的, 所以如果为空,
        # 很可能是因为 self.load_middleware() 没有调用成功.
        if self._request_middleware is None:
            with self.initLock:
                try:
                    # Check that middleware is still uninitialised.
                    if self._request_middleware is None:
                        因为 load_middleware() 可能没有调用, 调用一次.
                        self.load_middleware()
                except:
                    # Unload whatever middleware we got
                    self._request_middleware = None
                    raise
 
        set_script_prefix(base.get_script_name(environ))
        signls.request_started.send(sender=self.__class__) # __class__ 代表自己的类
 
        try:
            # 实例化 request_class = WSGIRequest, 将在日后文章中展开, 可以将其视为一个代表 http 请求的类
            request = self.request_class(environ)
 
        except UnicodeDecodeError:
            logger.warning('Bad Request (UnicodeDecodeError)',
                exc_info=sys.exc_info(),
                extra={
                    'status_code': 400,
                }
            )
            response = http.HttpResponseBadRequest()
        else:
            # 调用 self.get_response(), 将会返回一个相应对象 response<br>            ############# 关键的操作, self.response() 可以获取响应数据.          
            response = self.get_response(request)
 
        # 将 self 挂钩到 response 对象
        response._handler_class = self.__class__
 
        try:
            status_text = STATUS_CODE_TEXT[response.status_code]
        except KeyError:
            status_text = 'UNKNOWN STATUS CODE'
 
         # 状态码
        status = '%s %s' % (response.status_code, status_text)
 
        response_headers = [(str(k), str(v)) for k, v in response.items()]
 
        # 对于每个一个 cookie, 都在 header 中设置: Set-cookie xxx=yyy
        for c in response.cookies.values():
            response_headers.append((str('Set-Cookie'), str(c.output(header=''))))
 
        # start_response() 操作已经在上节中介绍了
        start_response(force_str(status), response_headers)
 
        # 成功返回相应对象
        return response
```
可以看出application的流程包括:加载所有中间件，以及执行框架相关的操作，设置当前线程脚本前缀，发送请求开始信号；处理请求，调用get_response()方法处理当前请求，该方法的的主要逻辑是通过urlconf找到对应的view和callback，按顺序执行各种middleware和callback。调用由server传入的start_response()方法将响应header与status返回给server。返回响应正文

### 2.2 django WSGI Server
负责获取http请求，将请求传递给WSGI application，由application处理请求后返回response。以Django内建server为例看一下具体实现。通过runserver运行django
项目，在启动时都会调用下面的run方法，创建一个WSGIServer的实例，之后再调用其serve_forever()方法启动服务。
```python
def run(addr, port, wsgi_handler, ipv6=False, threading=False): 
    server_address = (addr, port) 
    if threading: 
        httpd_cls = type(str('WSGIServer'), (socketserver.ThreadingMixIn, WSGIServer), {}) 
    else: 
        httpd_cls = WSGIServer # 这里的wsgi_handler就是WSGIApplication 
        httpd = httpd_cls(server_address, WSGIRequestHandler, ipv6=ipv6) 
    if threading: 
        httpd.daemon_threads = True httpd.set_app(wsgi_handler)    
    httpd.serve_forever()
```

下面表示WSGI server服务器处理流程中关键的类和方法。  

**WSGIServerrun()**方法会创建WSGIServer实例，主要作用是接收客户端请求，将请求传递给application，然后将application返回的response返回给客户端。
创建实例时会指定HTTP请求的handler：WSGIRequestHandler类
通过set_app和get_app方法设置和获取WSGIApplication实例wsgi_handler
处理http请求时，调用handler_request方法，会创建WSGIRequestHandler
实例处理http请求。
WSGIServer中get_request方法通过socket接受请求数据

**WSGIRequestHandler** 由WSGIServer在调用handle_request时创建实例，传入request、cient_address、WSGIServer三个参数，__init__方法在实例化同时还会调用自身的handle方法handle方法会创建ServerHandler实例，然后调用其run方法处理请求

**ServerHandler**  WSGIRequestHandler在其handle方法中调用run方法，传入self.server.get_app()参数，获取WSGIApplication，然后调用实例(__call__)，获取response，其中会传入start_response回调，用来处理返回的header和status。通过application获取response以后，通过finish_response返回response

**WSGIHandler**  WSGI协议中的application，接收两个参数，environ字典包含了客户端请求的信息以及其他信息，可以认为是请求上下文，start_response用于发送返回status和header的回调函数

虽然上面一个WSGI server涉及到多个类实现以及相互引用，但其实原理还是调用WSGIHandler，传入请求参数以及回调方法start_response()，并将响应返回给客户端