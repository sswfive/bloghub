---
title: django中类视图的使用
date: 2020-09-02 13:00:23
tags: 
  - Python
  - Django
---

# django中类视图的使用

##  一、类视图的引入：

- 函数视图：以函数的方式定义的视图；
  
    - 函数视图的缺点：一个视图对应的路径提供了多种不同HTTP请求方式的支持时，便需要在一个函数中编写不同的业务逻辑，代码可读性与复用性都不佳。
```python
 def register(request):
    """处理注册"""

    # 获取请求方法，判断是GET/POST请求
    if request.method == 'GET':
        # 处理GET请求，返回注册页面
        return render(request, 'register.html')
    else:
        # 处理POST请求，实现注册逻辑
        return HttpResponse('这里实现注册逻辑')
```
#### 因此Django中引用了类视图的概念

- 类视图：使用类来定义一个视图；
- 类视图的好处：
  
    - 代码可读性好；

    - 类视图相对于函数视图有更高的复用性，如果其他地方需要用到某个类视图的某个特定逻辑，直接继承该类视图即可；
```python
# 使用类视图可以将视图对应的不同请求方式以类中的不同方法来区别定义

from django.views.generic import View

class RegisterView(View):
    """类视图：处理注册"""

    def get(self, request):
        """处理GET请求，返回注册页面"""
        return render(request, 'register.html')

    def post(self, request):
        """处理POST请求，实现注册逻辑"""
        return HttpResponse('这里实现注册逻辑')
```

---

# 二、类视图的使用

流程：

**1. 定义一个类视图**；需要继承自Django提供的父类View，导入以下内容：
```python
from django.views.generic import View

或

from django.views.generic.base import View
```
**2. 配置路由**；——>使用类视图的as_view()方法来添加。

```python
urlpatterns = [
    # 视图函数：注册
    # url(r'^register/$', views.register, name='register'),
    # 类视图：注册
    url(r'^register/$', views.RegisterView.as_view(), name='register'),
]
```
# 三、 类视图原理

```python
@classonlymethod
    def as_view(cls, **initkwargs):
        """
        Main entry point for a request-response process.
        """
        ...省略代码...

        def view(request, *args, **kwargs):
            self = cls(**initkwargs)
            if hasattr(self, 'get') and not hasattr(self, 'head'):
                self.head = self.get
            self.request = request
            self.args = args
            self.kwargs = kwargs
            # 调用dispatch方法，按照不同请求方式调用不同请求方法
            return self.dispatch(request, *args, **kwargs)

        ...省略代码...

        # 返回真正的函数视图
        return view


    def dispatch(self, request, *args, **kwargs):
        # Try to dispatch to the right method; if a method doesn't exist,
        # defer to the error handler. Also defer to the error handler if the
        # request method isn't on the approved list.
        if request.method.lower() in self.http_method_names:
            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
        else:
            handler = self.http_method_not_allowed
        return handler(request, *args, **kwargs)
```
# 四、类视图使用装饰器

```python
类视图添加装饰器的有三种方法:

1 在URL配置中装饰器;
2 在类视图中装饰;
3 构造Mixin扩展类（使用面向对象多继承的特性）
```

#### 定义一个为函数视图准备的装饰器

```python
（在设计装饰器时基本都以函数视图作为考虑的被装饰对象），及一个要被装饰的类视图。

def my_decorator(func):
    def wrapper(request, *args, **kwargs):
        print('自定义装饰器被调用了')
        print('请求路径%s' % request.path)
        return func(request, *args, **kwargs)
    return wrapper

class DemoView(View):
    def get(self, request):
        print('get方法')
        return HttpResponse('ok')

    def post(self, request):
        print('post方法')
        return HttpResponse('ok')
```

## 1. 在URL配置中装饰

- 此种方式会为类视图中的所有请求方法都加上装饰器行为（因为是在视图入口处，分发请求方式前）。

- 缺点：因装饰行为被放置到了url配置中，单看视图的时候无法知道此视图还被添加了装饰器，不利于代码的完整性，不建议使用。

```python
urlpatterns = [
    url(r'^demo/$', my_decorate(DemoView.as_view()))
]

as_view是view类中的一个方法，
```

## 2 在类视图中装饰

- 在类视图中使用为函数视图准备的装饰器时，不能直接添加装饰器，需要使用method_decorator将其转换为适用于类视图方法的装饰器。

```python
from django.utils.decorators import method_decorator

# 为全部请求方法添加装饰器
class DemoView(View):

    # 如果需要对类中所有的方法都加装饰器，只需要把父类的dispatch方法重写，然后返回就可以了
    @method_decorator(my_decorator)
    def dispatch(self, *args, **kwargs):
        return super().dispatch(*args, **kwargs)

    def get(self, request):
        print('get方法')
        return HttpResponse('ok')

    def post(self, request):
        print('post方法')
        return HttpResponse('ok')


# 为特定请求方法添加装饰器
class DemoView(View):

    @method_decorator(my_decorator)
    def get(self, request):
        print('get方法')
        return HttpResponse('ok')

    def post(self, request):
        print('post方法')
        return HttpResponse('ok')
```

#### method_decorator装饰器还支持使用name参数指明被装饰的方法
```python
# 为全部请求方法添加装饰器
@method_decorator(my_decorator, name='dispatch')
class DemoView(View):
    def get(self, request):
        print('get方法')
        return HttpResponse('ok')

    def post(self, request):
        print('post方法')
        return HttpResponse('ok')


# 为特定请求方法添加装饰器
@method_decorator(my_decorator, name='get')
class DemoView(View):
    def get(self, request):
        print('get方法')
        return HttpResponse('ok')

    def post(self, request):
        print('post方法')
        return HttpResponse('ok')
```
#### 拓展：为什么需要使用method_decorator???

- 为函数视图准备的装饰器，其被调用时，第一个参数用于接收request对象
```python
def my_decorate(func):
    def wrapper(request, *args, **kwargs):  # 第一个参数request对象
        ...代码省略...
        return func(request, *args, **kwargs)
    return wrapper
```
- 而类视图中请求方法被调用时，传入的第一个参数不是request对象，而是self 视图对象本身，第二个位置参数才是request对象
```python
class DemoView(View):
    def dispatch(self, request, *args, **kwargs):
        ...代码省略...

    def get(self, request):
        ...代码省略...
```
- 所以如果直接将用于函数视图的装饰器装饰类视图方法，会导致参数传递出现问题。

- #### method_decorator的作用是为函数视图装饰器补充第一个self参数，以适配类视图方法。

- 如果将装饰器本身改为可以适配类视图方法的，类似如下，则无需再使用method_decorator。
```python
def my_decorator(func):
    def wrapper(self, request, *args, **kwargs):  # 此处增加了self
        print('自定义装饰器被调用了')
        print('请求路径%s' % request.path)
        return func(self, request, *args, **kwargs)  # 此处增加了self
    return wrapper
```

## 3 构造Mixin扩展类
> （使用面向对象多继承的特性）
使用Mixin扩展类，也会为类视图的所有请求方法都添加装饰行为

```python
# 使用扩展类的方法，实际上只需重写父类的as_view方法，并且传递不定长参数，
# 然后就能拿到入口的视图函数，在入口的视图函数中再添加装饰器，最后把装饰的结果当做返回值，
# 这样，就相当于调用的是自己的方法；

class MyDecoratorMixin(object):
    @classmethod
    def as_view(cls, *args, **kwargs):
        view = super().as_view(*args, **kwargs)
        view = my_decorator(view)
        return view

# class DemoView(Baseview，BaseView2，View)
class DemoView(MyDecoratorMixin, View):  # 在所有的类中的最后一个类都写View类，这样才能保证所有的都继承与object类
    def get(self, request):
        print('get方法')
        return HttpResponse('ok')

    def post(self, request):
        print('post方法')
        return HttpResponse('ok')
        
Django中的Mixin后缀表示的是扩展类
```
#### 拓展：关于在urls.py中路由到底调用的哪个类的as_view方法的详解：

所有的操作都是为了最后能调用最原生的View类中的as_view方法，并且要保证View中的as_view方法是最后一次被执行

1. 明确多继承中需找父类方法的顺序：


```python
条件：
    父类View  
    子类 BaseView, BaseView2    
    子子类：DemoView

继承顺序是：当调用DemoView的as_view方法时，首先去寻找出现在子类中的第一个类（BaseView）,然后去执行这个类的as_view方法，执行以后，
发现还需要去找as_view，此时，去寻找的是是子类中的同级类（BaseView2）,而不是View,找到以后，
然后再去执行BaseView2中的as_view方法，执行以后，此时才找到父类（View）中的as_view方法，最后返回的才是view函数；
——————————————————————————————————————————————————————————
条件：
    父类：View, View2
    子类：BaseView（继承于View）, BaseView2(继承于View2)
    子子类：DemoView
    
继承顺序是：当调用DemoView的as_view方法时，首先去寻找出现在子类中的第一个类（BaseView），
然后去执行这个类的as_view方法，执行以后，再去找上一级类中的as_view方法，此时找的是View,而不是BaseView2,
(因为BaseView和BaseView2不是继承与同一父类，)，找以后，执行as_view方法，然后返回view函数，并没有去执行BaseView2类中的方法，
```


   综上所述：为了能够调用所有类的as_view方法，必须将所有的类都继承于同一个父类，因此，把所有的扩展类都继承于（object基类），在所有的类中的最后一个类都写View类，这样才能保证所有的都继承与object类，然后才能保证调用是最原生的View的方法