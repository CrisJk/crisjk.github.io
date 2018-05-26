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

#### Python sql raw like

在使用原生sql时，用到%。只用一个%的话会报错

例如 `select * from table where date like "2016%" `

只需用两个%就能解决

`select * from table where date like "2016%%"





## Vue

#### vue Cannot read property 'get' of undefined

明显就是相关的属性没有引用。。。解决办法，在index.js中引入相关引用即可[https://github.com/pagekit/vue-resource/issues/441](https://github.com/pagekit/vue-resource/issues/441)



#### 在使用vue-for时尽可能提供key

为了便于 Vue 跟踪每个节点的身份，从而重新复用(reuse)和重新排序(reorder)现有元素，你需要为每项提供唯一的 `key` 属性，从而给 Vue 一个提示。理想的 `key` 值是每项都有唯一的 id

在使用 `v-for` 时，尽可能提供一个 `key`，除非迭代的 DOM 内容足够简单，或者你是故意依赖于默认行为来获得性能提升。

#### 序列化QuerySet

```python
    def Serialization(self,_obj: object) -> list:
        '''
        _obj: objext -> list, Python 3.6新加入的特性, 用来标识这个方法接收一个对象并返回一个list
        orm.raw序列化
        '''
        _list = []
        _get = []
        for i in _obj:
            _list.append(i.__dict__)

        for i in _list:
            del i['_state']
            _get.append(i)
        return _get
    
    subsidySet = Vegsubsidy.objects.raw('SELECT id,year,town,sum(totalMoney) from vegSubsidy WHERE year="2017" group by town')
     subsidySerializer=  self.Serialization(subsidySet)
        data = subsidySerializer
```

一种将QuerySet序列化的方法，应该有更好的办法.



#### 将vue-cli项目部署

vue-cli整合webpack的项目，部署到服务器上，若要用域名访问，则需要设置

```
this.disableHostCheck = true;
```

当然有更加安全的做法

[https://tonghuashuo.github.io/blog/webpack-dev-server-invalid-host-header.html](https://tonghuashuo.github.io/blog/webpack-dev-server-invalid-host-header.html)