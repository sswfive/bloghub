---
title: django中间件的使用
date: 2020-09-01 13:06:20
tags: 
  - Python
  - django
---
# django中间件的使用

## 中间件
> Django中的中间件和Flask中的请求钩子作用一样
>
> 使用中间件，在Django**处理视图的不同阶段**对输入或输出进行**干预**。

---

## 1 中间件的定义方法

>**新建一个middleware.py文件，这个文件可以在具体的应用中添加，也可以在全局领域中去添加；**
- 定义一个中间件工厂函数，然后返回一个可以调用的中间件。

- 中间件工厂函数需要接收一个可以调用的get_response对象。

- 返回的中间件也是一个可以被调用的对象，并且像视图一样需要接收一个request对象参数，返回一个response对象。
```python
def simple_middleware(get_response):
    # 此处编写的代码仅在Django第一次配置和初始化的时候执行一次。

    def middleware(request):
        # 此处编写的代码会在每个请求处理视图前被调用。

        response = get_response(request)

        # 此处编写的代码会在每个请求处理视图之后被调用。

        return response

    return middleware
```
#### 【Demo】
```python
# 例如，在users应用中新建一个middleware.py文件，

def my_middleware(get_response):
    print('init 被调用')
    def middleware(request):
        print('before request 被调用')
        response = get_response(request)
        print('after response 被调用')
        return response
    return middleware
    
    
# 定义一个视图进行测试

def demo_view(request):
    print('view 视图被调用')
    return HttpResponse('OK')
    
# 执行结果  

# init 被调用
# before request被调用
# view 视图被调用
# after response 被调用

# 注意：Django运行在调试模式下，中间件init部分有可能被调用两次。
```
- #### 定义好中间件后，需要在settings.py 文件中添加注册中间件
```python
# 在Django1.9版本之前，MIDDLEWARE叫做MIDDLEWARE_CLASSES

# 新版本的名称全部改为下面的名称了

#（以前定义中间件都是通过类的方式，现在全部都改为函数的方式去定义）

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    
    'users.middleware.my_middleware',  # 添加中间件
]
```
## 2 多个中间件的执行顺序

- 在请求视图被处理前，中间件由上至下依次执行

- 在请求视图被处理后，中间件由下至上依次执行

#### 【Demo】

- **定义两个中间件**
```python
def my_middleware(get_response):
    print('init 被调用')
    def middleware(request):
        print('before request 被调用')
        response = get_response(request)
        print('after response 被调用')
        return response
    return middleware

def my_middleware2(get_response):
    print('init2 被调用')
    def middleware(request):
        print('before request 2 被调用')
        response = get_response(request)
        print('after response 2 被调用')
        return response
    return middleware
 
 
# 执行结果如下：（执行之前先需要在MIDDLEWARE中添加注册）

#init2 被调用
#init 被调用
#before request 被调用
#before request 2 被调用
#view 视图被调用
#after response 2 被调用
#after response 被调用÷÷
```
- 在settings.py中注册添加两个中间件

```python
MIDDLEWARE = [
    '……'
    
    'users.middleware.my_middleware',  # 添加
    'users.middleware.my_middleware2',  # 添加
]
```

