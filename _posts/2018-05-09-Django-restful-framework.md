---
author: Kuang
title: Django restful framework + vue.js前后端分离学习过程中遇到的坑
categories: web
tags: django

---

> 学习入门材料，推荐https://github.com/twtrubiks/django-rest-framework-tutorial





## Django 

#### Django Restful framework Routers

第一个坑点出现在，更新后的框架取消了一些东西，例如`from rest_framework.decorators import detail_route, list_route` 取消掉了，更改为`from rest_framework.decorators import action`

当你代码中写下

```
@detail_route(methods=['get'])
def detail(self, request, pk=None):
```

时，会报错

```
Traceback (most recent call last):
  File "/usr/local/lib/python3.6/dist-packages/django/core/handlers/exception.py", line 35, in inner
    response = get_response(request)
  File "/usr/local/lib/python3.6/dist-packages/django/core/handlers/base.py", line 128, in _get_response
    response = self.process_exception_by_middleware(e, request)
  File "/usr/local/lib/python3.6/dist-packages/django/core/handlers/base.py", line 126, in _get_response
    response = wrapped_callback(request, *callback_args, **callback_kwargs)
  File "/usr/local/lib/python3.6/dist-packages/django/views/decorators/csrf.py", line 54, in wrapped_view
    return view_func(*args, **kwargs)
  File "/usr/local/lib/python3.6/dist-packages/rest_framework/viewsets.py", line 103, in view
    return self.dispatch(request, *args, **kwargs)
  File "/usr/local/lib/python3.6/dist-packages/rest_framework/views.py", line 483, in dispatch
    response = self.handle_exception(exc)
  File "/usr/local/lib/python3.6/dist-packages/rest_framework/views.py", line 443, in handle_exception
    self.raise_uncaught_exception(exc)
  File "/usr/local/lib/python3.6/dist-packages/rest_framework/views.py", line 480, in dispatch
    response = handler(request, *args, **kwargs)
TypeError: 'bool' object is not callable
```

解决办法:更改函数名detail成其它名字



## Vue

#### vue Cannot read property 'get' of undefined

明显就是相关的属性没有引用。。。解决办法，在index.js中引入相关引用即可[https://github.com/pagekit/vue-resource/issues/441](https://github.com/pagekit/vue-resource/issues/441)