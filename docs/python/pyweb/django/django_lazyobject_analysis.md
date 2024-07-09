---
title: django源码学习之LazyObject
date: 2021-10-06 12:50:19
tags: 
  - Python
  - Django
---

### 导入
```
from django.utils.functional import LazyObject
```

### LazyObject源码分析

```
empty = object()  # empty是一个object对象，表示还未初始化

"""
#为类方法实现通用的惰性加载方法，即查看对象是否实例化
#若还未实例化，调用子类实例化方法
#最后在生成的实例上调用原来的func类方法
"""
def new_method_proxy(func):  
# new_method_proxy是个工厂函数，为其他函数增加实例化的功能，
    def inner(self, *args):
        # 判断是否empty,表示为空，未实例化
        if self._wrapped is empty:
            # 调用继承类实现的_setup,实例化类
            self._setup()
        return func(self._wrapped, *args)
    return inner


class LazyObject(object):
    """
    A wrapper for another class that can be used to delay instantiation of the
    wrapped class.

    By subclassing, you have the opportunity to intercept and alter the
    instantiation. If you don't need to do that, use SimpleLazyObject.
    """

    # Avoid infinite recursion when tracing __init__ (#19456).
    _wrapped = None

    def __init__(self):
        # Note: if a subclass overrides __init__(), it will likely need to
        # override __copy__() and __deepcopy__() as well.
        self._wrapped = empty  #标记是否有实例化

     # 通过工厂函数生成__getattr__方法，为其判断是否实例化，如果未实例化，则调用_setup实例化类
    __getattr__ = new_method_proxy(getattr)  

    def __setattr__(self, name, value):
        if name == "_wrapped": #若修改的为_wrapped，特殊处理
            # Assign to __dict__ to avoid infinite __setattr__ loops.
            self.__dict__["_wrapped"] = value
        else:
            if self._wrapped is empty:
                self._setup()
            setattr(self._wrapped, name, value)   #实例化后，修改实例的属性

    def __delattr__(self, name):
        if name == "_wrapped":  #不可删除_wrapped属性
            raise TypeError("can't delete _wrapped.")
        if self._wrapped is empty:
            self._setup()
        delattr(self._wrapped, name)

    # 需要子类实现自己的_setup
    def _setup(self):
        """
        Must be implemented by subclasses to initialize the wrapped object.
        """
        #只可由子类调用
        raise NotImplementedError('subclasses of LazyObject must provide a _setup() method')

    # Because we have messed with __class__ below, we confuse pickle as to what
    # class we are pickling. We're going to have to initialize the wrapped
    # object to successfully pickle it, so we might as well just pickle the
    # wrapped object since they're supposed to act the same way.
    #
    # Unfortunately, if we try to simply act like the wrapped object, the ruse
    # will break down when pickle gets our id(). Thus we end up with pickle
    # thinking, in effect, that we are a distinct object from the wrapped
    # object, but with the same __dict__. This can cause problems (see #25389).
    #
    # So instead, we define our own __reduce__ method and custom unpickler. We
    # pickle the wrapped object as the unpickler's argument, so that pickle
    # will pickle it normally, and then the unpickler simply returns its
    # argument.
    def __reduce__(self):
        if self._wrapped is empty:
            self._setup()
        return (unpickle_lazyobject, (self._wrapped,))

    def __copy__(self):
        if self._wrapped is empty:
            # If uninitialized, copy the wrapper. Use type(self), not
            # self.__class__, because the latter is proxied.
            return type(self)()
        else:
            # If initialized, return a copy of the wrapped object.
            return copy.copy(self._wrapped)

    def __deepcopy__(self, memo):
        if self._wrapped is empty:
            # We have to use type(self), not self.__class__, because the
            # latter is proxied.
            result = type(self)()
            memo[id(self)] = result
            return result
        return copy.deepcopy(self._wrapped, memo)

    __bytes__ = new_method_proxy(bytes)
    __str__ = new_method_proxy(str)
    __bool__ = new_method_proxy(bool)

    # Introspection support
    __dir__ = new_method_proxy(dir)

    # Need to pretend to be the wrapped class, for the sake of objects that
    # care about this (especially in equality tests)
    __class__ = property(new_method_proxy(operator.attrgetter("__class__")))
    __eq__ = new_method_proxy(operator.eq)
    __lt__ = new_method_proxy(operator.lt)
    __gt__ = new_method_proxy(operator.gt)
    __ne__ = new_method_proxy(operator.ne)
    __hash__ = new_method_proxy(hash)

    # List/Tuple/Dictionary methods support
    __getitem__ = new_method_proxy(operator.getitem)
    __setitem__ = new_method_proxy(operator.setitem)
    __delitem__ = new_method_proxy(operator.delitem)
    __iter__ = new_method_proxy(iter)
    __len__ = new_method_proxy(len)
    __contains__ = new_method_proxy(operator.contains)
```
### 总结：
1. LazyObject的核心逻辑，初始化的时候把_wrapped设置为empty，未对实际的类进行实例化操作，达到延迟类实例化的效果。只有对结果需要进行某些操作的时候，比如获取属性，才会去调用_setup进行实例化。而且_setup必须由子类实现。
2. 如果想要让某个类有延迟实例化的功能，必须做两件事情，1）继承LazyObject；2）实现_setup方法。

### Demo示例
```
# 有延迟实例化需求的类DemoObject
class DemoObject(object):  
    def __init__(self):  
        self.title = 'just a demo'  

# 因为继承了LazyObject，所以DemoLazyObject有延迟实例化的功能
class DemoLazyObject(LazyObject):
    # 子类需要重新实现_setup方法，该方法实例化类DemoObject，并把生成的对象赋给_wrapped。
    def _setup(self):
        self._wrapped = DemoObject()  

# 实例化类DemoLazyObject，生成对象a_demo，此时并没有实际实例化类DemoObject。
a_demo = DemoLazyObject()
# 调用a_demo.title的时候，会触发类DemoObject的实例化
print(a_demo.title)
```
代码运行流程：

    当实例化类DemoLazyObject时，并没有真正去实例化类 DemoObject，它的实例化在_setup里面。只有执行了后面那行print(a_demo.title)，才会真正触发类 DemoObject的实例化。

    第一次的调用流程是print(a_demo.title)->LazyObject.__getattr__->new_method_proxy(getattr)->DemoLazyObject._setup(self)->DemoObject.__init__(self)->LazyObject.getattr(self._wrapped, *args)。

    以后再调用的时候就不会进入DemoLazyObject._setup(self)和DemoObject.__init__(self)，而是直接返回LazyObject.getattr(self._wrapped, *args)。


---
### django项目中的应用

**情景需求：调用oss的api去实现将压缩包文件上传到oss的private bucket中**
- models:
```
file_path = models.FileField(u'文件路径(*.tar.gz)',upload_to=generate_package_filename, max_length=1024,storage=system_package_storage)

FileField:字符串类型，路径保存到数据库，文件上传到指定目录
upload_to:上传文件的保存路径
storage:存储组件，默认是django.core.files.storage.FileSystemStorage
```
- 生成上传文件的保存路径的方法
```
def generate_package_filename(instance, filename):
    return {'product_key': instance.product.product_key}, 'system_package/%d/%s/%s%s' % (
        instance.device_type, instance.version, int(time.mktime(datetime.now().timetuple())), file_extension(filename))
```
- system_package_storage的实现

```
class AliyunBaseStorage(Storage):  # 继承from django.core.files.storage import Storage
    """
    Aliyun OSS2 Storage
    """

    private = False

    def __init__(self):
        self.kwargs = {}

    '''
    ' 根据model中定义的upload_to函数返回值，取出文件存储路径
    ' 默认直接传一个文件名，
    ' 跟product相关的会传product相关属性过来
    '''
    def get_available_name(self, upload_to):
        super(AliyunBaseStorage, self).get_available_name(upload_to)

    def get_iot_service(self):
        kwargs = self.kwargs
        if not kwargs.has_key('product_name') and not kwargs.has_key('product_key'):
            kwargs.update({ 'product_name' : 'wastebin' })
        return AliyunIoTService(**kwargs)

    def get_bucket_read(self):
        iot_service = self.get_iot_service()
        if self.private:
            return iot_service.get_private_bucket(aliyun_sts_type=AliyunSTSType.STSReadOnly)
        return iot_service.get_media_bucket(aliyun_sts_type=AliyunSTSType.STSReadOnly)

    def get_bucket_write(self):
        iot_service = self.get_iot_service()
        if self.private:
            return iot_service.get_private_bucket(aliyun_sts_type=AliyunSTSType.STSWriteOnly)
        return iot_service.get_media_bucket(aliyun_sts_type=AliyunSTSType.STSWriteOnly)

    def get_location(self):
        return ALIYUN_SERVER_TYPE.lower()

    def _clean_name(self, name):
        clean_name = posixpath.normpath(name).replace('\\', '/')

        if name.endswith('/') and not clean_name.endswith('/'):
            return clean_name + '/'
        else:
            return clean_name

    def _normalize_name(self, name):
        base_path = force_text(self.get_location())
        base_path = base_path.rstrip('/')

        final_path = urljoin(base_path.rstrip('/') + "/", name)

        base_path_len = len(base_path)
        if (not final_path.startswith(base_path) or
                final_path[base_path_len:base_path_len + 1] not in ('', '/')):
            raise SuspiciousOperation("Attempted access to '%s' denied." %
                                      name)
        return final_path.lstrip('/')

    def _get_target_name(self, name):
        name = self._normalize_name(self._clean_name(name))
        if six.PY2:
            name = name.encode('utf-8')
        return name

    def _open(self, name, mode='rb'):
        return AliyunFile(name, self, mode)

    def _save(self, name, content):
        # 为保证django行为的一致性，保存文件时，应该返回相对于`media path`的相对路径。
        target_name = self._get_target_name(name)

        content.open()
        content_str = b''.join(chunk for chunk in content.chunks())
        self.get_bucket_write().put_object(target_name, content_str)
        content.close()

        return self._clean_name(name)

    def get_file_header(self, name):
        name = self._get_target_name(name)
        return self.get_bucket_read().head_object(name)

    def exists(self, name):
        return self.get_bucket_read().object_exists(name)

    def size(self, name):
        file_info = self.get_file_header(name)
        return file_info.content_length

    def modified_time(self, name):
        file_info = self.get_file_header(name)
        return datetime.datetime.fromtimestamp(file_info.last_modified)

    def listdir(self, name):
        name = self._normalize_name(self._clean_name(name))
        if name and name.endswith('/'):
            name = name[:-1]

        files = []
        dirs = set()

        for obj in ObjectIterator(self.get_bucket_read(), prefix=name, delimiter='/'):
            if obj.is_prefix():
                dirs.add(obj.key)
            else:
                files.append(obj.key)

        return list(dirs), files

    def url(self, name):
        name = self._normalize_name(self._clean_name(name))
        # name = filepath_to_uri(name) # 这段会导致二次encode
        name = name.encode('utf8')
        # 做这个转化，是因为下面的_make_url会用urllib.quote转码，转码不支持unicode，会报错，在python2环境下。
        return self.get_bucket_read()._make_url(self.get_bucket_read().bucket_name, name)

    def read(self, name):
        pass

    def delete(self, name):
        name = self._get_target_name(name)
        result = self.get_bucket_write().delete_object(name)
        if result.status >= 400:
            raise AliyunOperationError(result.resp)


class AliyunSystemPackageStorage(AliyunBaseStorage):

    private = True

    def _save(self, name, content):
        super(AliyunSystemPackageStorage, self)._save(name, content)
        return name

    def get_available_name(self, upload_to):
        kwargs, name = upload_to
        self.kwargs.update(kwargs)

        dir_name, file_name = os.path.split(name)
        file_root, file_ext = os.path.splitext(file_name)
        count = itertools.count(1)
        while self.exists(name):
            name = os.path.join(dir_name, "%s_%s%s" % (file_root, next(count), file_ext))

        return name

class OSSSystemPackageStorage(LazyObject):
    def _setup(self):
        self._wrapped = AliyunSystemPackageStorage()

system_package_storage = OSSSystemPackageStorage()
```
