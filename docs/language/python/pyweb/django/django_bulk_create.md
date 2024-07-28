---
title: django批量导入数据
date: 2020-11-14 22:07:39
tags: 
  - Python
  - Django
---

# django批量导入数据

- 在Django中需要向数据库中插入多条数据（list）。使用如下方法，每次save()的时候都会访问一次数据库。导致性能问题:

```python
for i in resultlist:
    p = Account(name=i) 
    p.save()

```
- 在django1.4以后加入了新的特性。使用django.db.models.query.QuerySet.bulk_create()批量创建对象，减少SQL查询次数。改进如下：


```python
querysetlist=[]
for i in resultlist:
    querysetlist.append(Account(name=i))        
Account.objects.bulk_create(querysetlist)

```

- Model.objects.bulk_create() 更快更方便
    - 常规用法:
    
    ```python
    #!/usr/bin/env python
    #coding:utf-8
     
    import os
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
     
    '''
    Django 版本大于等于1.7的时候，需要加上下面两句
    import django
    django.setup()
    否则会抛出错误 django.core.exceptions.AppRegistryNotReady: Models aren't loaded yet.
    '''
     
    import django
    if django.VERSION >= (1, 7):#自动判断版本
        django.setup()
     
     
    def main():
        from blog.models import Blog
        f = open('oldblog.txt')
        for line in f:
            title,content = line.split('****')
            Blog.objects.create(title=title,content=content)
        f.close()
     
    if __name__ == "__main__":
        main()
        print('Done!')
    
    ```
    
    - 使用批量导入:
    
    ```python
    #!/usr/bin/env python
    import os
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
     
    def main():
        from blog.models import Blog
        f = open('oldblog.txt')
        BlogList = []
        for line in f:
            title,content = line.split('****')
            blog = Blog(title=title,content=content)
            BlogList.append(blog)
        f.close()
         
        Blog.objects.bulk_create(BlogList)
     
    if __name__ == "__main__":
        main()
        print('Done!')
    ```
    
- 由于Blog.objects.create()每保存一条就执行一次SQL，而bulk_create()是执行一条SQL存入多条数据，做会快很多！当然用列表解析代替 for 循环会更快！！

```python
#!/usr/bin/env python
import os
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")
 
def main():
    from blog.models import Blog
    f = open('oldblog.txt')
     
    BlogList = []
    for line in f:
        parts = line.split('****')
        BlogList.append(Blog(title=parts[0], content=parts[1]))
     
    f.close()
         
    # 以上四行 也可以用 列表解析 写成下面这样
    # BlogList = [Blog(title=line.split('****')[0], content=line.split('****')[1]) for line in f]
     
    Blog.objects.bulk_create(BlogList)
 
if __name__ == "__main__":
    main()
    print('Done!')
```

- 批量导入时数据重复的解决方法
>如果你导入数据过多，导入时出错了，或者你手动停止了，导入了一部分，还有一部分没有导入。或者你再次运行上面的命令，你会发现数据重复了，怎么办呢？

- django.db.models 中还有一个函数叫 get_or_create(),之前文章中也提到过,有就获取过来，没有就创建，用它可以避免重复，但是速度可以会慢些，因为要先尝试获取，看看有没有

只要把上面的:
```python
Blog.objects.create(title=title,content=content)
```
- 换成下面的就不会重复导入数据了

```python
Blog.objects.get_or_create(title=title,content=content)
```

- 返回值是(BlogObject, True/False)新建时返回 True, 已经存在时返回 False。