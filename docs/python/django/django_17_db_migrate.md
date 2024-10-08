---
title: django1.7以前的版本执行数据库迁移
date: 2020-08-21 22:08:16
tags: 
  - Python
  - Django
---

# django1.7以前的版本执行数据库迁移

##  django1.7以前的版本执行数据库迁移
```python
#  1, 安装South
 pip install South
 
# 2.在settings.py的INSTALL_APPS中添加：
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',
    'south',
)

# 3.执行以下命令，Django会新建一个 south_migrationhistory 表，用来记录数据表更改(Migration)的历史纪录
 python manage.py syncdb
 
$ python manage.py syncdb
Syncing...
Creating tables ...
Creating table south_migrationhistory
Installing custom SQL ...
Installing indexes ...
No fixtures found.
 
Synced:
 > django.contrib.admin
 > django.contrib.auth
 > django.contrib.contenttypes
 > django.contrib.sessions
 > django.contrib.messages
 > django.contrib.staticfiles
 > blog
 > south
 
Not synced (use migrations): 

# 4.当models.py做任何修改，只要执行下面操作
python manage.py schemamigration 应用名 --auto

# 5.执行迁移：
python manage.py migrate

备注：上述操作的参考链接：https://code.ziqiangxuetang.com/django/django-schema-migration.html
```
