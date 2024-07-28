---
title: django中ORM查询方法总结
date: 2020-11-06 22:07:59
tags: 
  - Python
  - Django
---

# django中ORM查询方法总结

##  **查询**

查询是通过ORM设置好的API,也就是各种方法和属性实现的.查询分为查询方法和查询条件.

## **查询方法 [Methods that return new QuerySets](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#methods-that-return-new-querysets)**

查询方法返回的都是QuerySet对象(包含一组对象的列表)或者具体的对象(表示一行数据),来看一下基本的查询方法:

|     ORM一般查询     |                                                              |
| :-----------------: | :----------------------------------------------------------: |
|        方法         |                             解释                             |
|        all()        | 查询所有结果.objects后边的.all()可以省略,省略的时候就是对所有结果操作. |
|   get(**kwargs):    | 返回与所给筛选条件相匹配的对象，返回结果有且只有一个，如果符合筛选条件的对象超过一个或者没查找到都会抛出错误。返回具体对象,不是QuerySet |
|  filter(**kwargs):  | 返回与所给筛选条件相匹配的结果集(QuerySet对象,放有所有查询结果的列表),即使结果只有一个,也是返回一个结果集,需要从中取出每条数据处理.如果没有结果,返回空集.以后多用filter,基本不用get |
|  exclude(**kwargs)  |              不符合筛选条件的结果.返回QuerySet               |
|   values(*field)    | 返回一个QuerySet,,每一个元素是代表一行数据的一个字典,里边用键=字段名 值= 数据 的方式存放所有查询到的结果.可以加参数指定具体字段,多个字段名用逗号分隔,不指定参数则取出所有字段.只能作用在QuerySet对象上,不能作用于单独的对象. |
| values_list(*field) | 和values类似,只不过QuerySet的每一个元素是一个元组,按参数顺序放着字段的值.只能作用在QuerySet对象上,不能作用于单独的对象. |
|  order_by(*field)   | 按照指定的字段排序.在字符串开头写-表示倒序如”-price”.不使用order_by的话,默认会按照META里设置的顺序对结果排序.返回QuerySet |
|      reverse()      | reverse只能在指定顺序的QuerySet上调用,比如用过了order_by方法或者META里写了排序方法.得到反向排序的顺序.返回QuerySet |
|     distinct()      |   从返回结果中剔除重复纪录.跨表查询的时候经常用此方法去重.   |
|       count()       | 返回数据库中匹配查询结果集(QuerySet)中的对象数量.是一个Int值. |
|       first()       |    返回QuerySet中第一个对象.返回的是具体对象,不是QuerySet    |
|       last()        |   返回QuerySet中最后一个对象.返回的是具体对象,不是QuerySet   |
|      exists()       | 返回一个布尔值.如果QuerySet包含数据，就返回True，否则返回False.用于快速判断是否查询到结果. |

## **查询条件 Field lookups**

基本查询语句的参数是**kwargs的都是各种查询条件,上边例子里是用的=来表示条件,这是精确匹配,还可以采用其他条件.这部分对应的都是MySQL的WHERE语句后边的内容.在Django ROM里叫做 [Field lookups](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#field-lookups).这些Field lookups 关键字都用双下划线加在字段名后边.有多个条件的,将条件用逗号隔开放在上边的基本查询语句中,比如`models.Person.objects.filter(id__gt=2,id__lt=5)`.

| Field lookups 查询条件 |                                                              |
| :--------------------: | :----------------------------------------------------------: |
|          条件          |                             解释                             |
|           =            |              表示精确匹配,只选择符合条件的对象               |
|         iexact         |         大小写不敏感的精确匹配,常用来匹配一段字符串          |
|        contains        |            包含有某段字符串内容的匹配,大小写敏感             |
|       icontains        |                   大小写不敏感的icontains                    |
|           in           |                在一个列表内.和SQL语句的in一样                |
|           gt           |                             大于                             |
|          gte           |                           大于等于                           |
|           lt           |                             小于                             |
|          lte           |                           小于等于                           |
|       startswith       |                   以字符串开始,大小写敏感                    |
|      istartswith       |                 大小写不敏感版本的startswith                 |
|        endswith        |                   以字符串结尾,大小写敏感                    |
|       iendswith        |                  大小写不敏感版本的endswith                  |
|         range          | 是否在一个范围内,包含范围的两端,=号后边是一个两个元素的元组,表示开始和结束 |
|          date          | 用在datetime类别的字段上,单独取出日期(datetime对象)来比较,还可以后续接其他Field lookups,详细看[这里](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#date) |
|          year          |      和date一样,只不过单独取出来年份比较.是一个INT类型       |
|         month          |    和date一样,只不过单独取出来月份进行比较,是一个INT类型     |
|          day           | 和date一样,只不过单独取出来日进行比较,是一个INT类型.year,month和day经常用在快速过滤日期,比使用datetime对象比较要简单很多. |
|          week          |   和date一样,只不过单独取出来第几周进行比较,是一个INT类型.   |
|        week_day        | 和date一样,只不过单独取出来星期几日进行比较,是一个INT类型.星期天是1,星期六是7.比较常用. |
|          time          | 和date一样,只不过单独取出来时间进行比较,是一个 datetime.time对象. |
|          hour          |  从datetime和time fileds里取小时,范围是0-23,是一个INT类型.   |
|         minute         |             同hour,取分钟,范围0-59,是一个INT类型             |
|         second         |              同hour,取秒,范围0-59,是一个INT类型              |
|         isnull         |                      判断字段是否是Null                      |
|         regex          |    大小写敏感的正则匹配模式,等号后边需要是一个正则表达式     |
|         iregex         |                   大小写不敏感的正则表达式                   |

## **跨表查询**

跨表查询指的是依靠关系进行查询的方法.其核心就是依据外键以及一对一,多对多字段进行的查找.先来看经典的外键一对多查询:

### **ForeignKey 一对多查询**

先来了解一下正向查询和反向查询
**正向查询**:外键所在的表去查外键关联的表.
**逆向查询**:一个表去查另外一个外键关联到当前这个表的表.
例子:

```python
class Publisher(models.Model):
    id = models.AutoField(primary_key=True)
    name = models.CharField(max_length=32, unique=True)

    def __str__(self):
        return "我是一个出版社对象:{}".format(self.name)


class Book(models.Model):
    id = models.AutoField(primary_key=True)
    price = models.DecimalField(max_digits=5, decimal_places=2, default=99.99)
    # 库存数
    kucun = models.IntegerField(default=1000)
    # 卖出数
    maichu = models.IntegerField(default=0)

    title = models.CharField(max_length=32)
    # 外键
    # related_name="books" 反向查询是用来代替 book_set的
    publisher = models.ForeignKey(
        to="Publisher",
        on_delete=models.CASCADE)

    def __str__(self):
        return self.title
```

从Book表去查Publisher查叫做正向查询,从Publisher表去查Book叫做反向查询.

正向查询,在当前表内查到结果之后,直接调用外键对应的属性,就可以得到指向的外键对象.由于这里是一对多的,正向查询回去的结果就是对应的对象,不是QuerySet.例子:

```python
book_obj = models.Book.objects.all().first()
ret = book_obj.publisher  # 和我这本书关联的出版社对象
print(ret, type(ret))
```

结果直接就是书对应的出版社对象.如果想查对应的出版社的名称,由于查到了对象,可以再取名称,但有简单的方法来查外键对应的具体字段:

```python
ret = models.Book.objects.filter(id=1).values_list("publisher__name")
print(ret)
```

双下划线就表示跨了表,前边的publisher是表名,后边是表示字段名.由于是拿到具体值,所以只能在values和values_list里应用该方法.

反向查询,如果要查一个出版社所有的书,有个特殊的方法是表名+_set来反着找”一对多”里的”多”,来看例子:

```python
publisher_obj = models.Publisher.objects.first()  # 找到第一个出版社对象
books = publisher_obj.book_set.all()  # 找到第一个出版社出版的所有书
titles = books.values_list("title")
```

这里的book是表名,后边加上_set就表示来反着找,结果是一个QuerySet,可以把book表里由第一个出版社出版的所有书都找到.
如果在book表的外键里给外键指定了related_name,那么此时就不要用表名,直接用related_name的名字就可以了.比如指定了related_name为all_books.上边例子的第二句话就可以改写成:

```python
books = publisher_obj.allbook.all()
```

注意,如果起了外键别名,再使用原表名_set会报错.实际上别名代替了原来对象上的表名_set方法.
同样,也可以直接查字段,也是用双下划线来连接表名和字段.

```python
ret = models.Publisher.objects.filter(id=1).values_list("book__title")
print(ret)
```

book是表名,title是字段名.注意,这里如果外键指定了related_name的值而没有指定related_query_name的值,则需要将book换成其related_name的值,如果同时指定了related_name和related_query_name的值,则需要换成related_query_name的值,如果只指定了related_query_name的值,也需要换成其值.
结果得到的是元素为元组的列表,每个元组内就是查询字段的结果.

### **ManyToManyField 多对多查询**

在ORM中,多对多以及外键的反向查询(结果集都是取到多个对象的时候),内部都是通过一个叫做 RelatedManager类的实例,即关联上下文管理器来操作的.
比如在外键的例子中,如果打印`publisher_obj.allbook`,可以看到结果是一个 RelatedManager类.针对这个类,像.all()就是取全部结果,除此之外,针对这个类还有一些特殊的操作:

**`create()`** 用于从创建一对多关系

```python
models.Author.objects.first().book_set.create(title="番茄物语", publish_date=datetime.date.today())
```

通过Author.objects.first()取到一个具体的作者对象,然后.book_set取到上下文管理器对象,然后对这个对象进行.create方法,结果是先在book表内建立一本书,然后在多对多的表内建立这本书与这个作者的关系.

**`add()`** 操作一个已经存在的对象,将对象添加到对应关系里

```python
new_book = models.book.objects.create(name= '新的书').save()  # 在book表内建立一本新书
models.Author.objects.first().book_set.add(new_book)  # 在多对多表内添加一行,记录第一个作者和这本新书的对应关系.
```

add的参数可以是对象,可以是解开的QeurySet对象(*ret),也可以是一个数字表示id或者拆解开的列表id,如add(*[3,12,19]).

**`set()`** 一次性在多对多表里设置一组对应关系
在之前的项目中已经使用过set了,其结果就是将对应的一组id或者对象代表的内容,在多对多表里去当前对象联系起来/

```python
book_obj = models.Book.objects.first()
book_obj.authors.set([2, 3])
```

这个例子就是将第一本书与id为2,3的作者在多对多表里联系起来.

**`remove()`** 移除参数指定的对象与当前对象的对应关系

```python
book_obj = models.Book.objects.first()
book_obj.authors.remove(3)
```

remove接受的参数和add一样.

**`clear()`** 表示清除当前对象在多对多表里对应某个表的全部对应关系.

```python
book_obj = models.Book.objects.first()
book_obj.authors.clear()
```

这个操作的意思表示移除第一本书和所有作者的关联关系.也就是在多对多表里删除所有book_id = 1的数据行.

由于上下文管理器在外键的反向查询的过程中也存在,因此也是可以用上边五种方法来操作的,只不过在外键反向查询中,不存在多对多的表格.所以clear()和remove()操作实际上是将外键那一列修改成Null,因此必须要求外键设置成Null=True,否则会报错.

### **一对一查询**

一对一的关系字段是models.OneToOneField(to=).实际上正查反查都是只能出来一条字段.不再赘述.其实用id也可以取得相同的数据.
[这里](https://www.cnblogs.com/pythonxiaohu/p/5814247.html)有一篇文章讲了Django中关系的总结.

## **使用聚合函数与分组查询**

在ORM中,一样可以使用便利的聚合函数以及非常常见的分组查询.

### **聚合函数**

聚合函数的方法是`aggregate()`,在使用这个方法之后,查询语句就结束了,这个方法返回一个字典,里边包含着使用聚合函数的结果.
在使用聚合函数之前,必须导入对应的聚合函数模块:

```
from django.db.models import Avg, Sum, Max, Min, Count
```

这些函数的参数都是字段名称.对于生成的结果字典,如果没有指定名称,则Django会自动设置一个名称.如果指定了名称,就会使用指定的名称,然后再从字典里取数即可:

```python
models.Book.objects.all().aggregate(Avg("price"))
# 结果是{'price__avg': 13.233333},其中的"price__avg"由Django自动生成

models.Book.objects.aggregate(average_price=Avg('price'))
# 结果是{'average_price': 13.233333}, 其中的"average_price"是自定义的名称.

models.Book.objects.all().aggregate(Avg("price"), Max("price"), Min("price"))
# 同时使用多个聚合函数,可以用逗号分割,结果字典中会包括全部结果.
```

### **分组**

分组的方法是`.annotate()`.这里可能有点抽象,其实annotate的作用就是先按照之前的查找方式返回数据,再将返回的多行数据按照某个指定的字段进行分组操作,然后将分组过的每一行结果都包在一个QuerySet中返回.其中.annotate之前的部分如果是通过外键查询,其本身就相当于连表操作,再分组之后就是分组后的QuerySet了.
比如在自己编写的行政物品管理系统内,需要列出每一种物品的结余,即通过物品列表查到每一种物品对应的所有出入库记录,再将出库和入库记录求和分别求和并相减.
用SQL语句是先通过move的item_id连表,然后按照item_id分组并对in_num和out_num字段进行求和再相减.
在ORM里,通过物品列表的这一段写在前边,到外键所在的表去查询,则将外键所在的表及字段名称写在annotate内部的聚合函数内,即像这样:

```python
ret = models.Items.objects.all().annotate(num_balance=Sum("moves__in_num") - Sum("moves__out_num"))
```

前边的`ret = models.Items.objects.all()`表示按照所有的物品分组.由于是正向查询即通过物品查余额,所以`moves__in_num`和`moves__out_num`其中采取表名__字段名的写法.`.annotate`就表示按照之前的对象分组并且按照参数中的某个字段进行处理,这里是分组以后对分组的in_num求和再减去out_num的求和,就得到了按照物品分组的余额.这里是反向操作,直接使用小写的类名即可,不用加__set,如果是正向,用外键也可以.(当然一般分组不会用正向查询)
ret是一个QuerySet,其中的每一个元素就是一个对象,表示一行经过分组过后的数据,这一行里的属性包括Items表里的全部属性,以及自定义的num_balance属性.

这里直接就使用了跨表查询,如果想在同一个表内,按照某一个字段进行分组,像上边例子里的`models.Items.objects.all()`的部分,就需要改成当前表里字段的用values的查询结果,也就是告诉ORM按照哪个字段来分组.
在自己编写的进销存系统中,查询moves表按照item_id分类的in_sum字段的和:

```python
ret = models.Moves.objects.values("item_id").annotate(insum = Sum("in_num"))
```

这个就是先用了item_id过滤出内容,结果的QuerySet里只包括item_id字段的值,然后在对其进行分组并对对应的in_num字段求和,求和的结果放到每一个结果里的insum字段名中.
ret实际上就是一个分组并且计算过的表的每一行内容的QuerySet,再从里边取出每一行进行操作和展示即可.

分组的操作不是太直观,用[这里](http://www.cnblogs.com/liwenzhou/p/8660826.html)分组的例子举一个例子:
在一张表内进行分组,用SQL语句比较直观:

```python
select dept,AVG(salary) from employee group by dept;
```

如果换成ORM,首先明确是要查那张表.这个查询只在一个表内查询,查询的结果包括dept和AVG(salary),group by 后边是dept.
这说明按照dept先分组,然后对于聚合后的结果,用avg对salary求平均数,最后返回分组后的部门和结果.

#### **SQL语句与ORM的对应关系:**

**第一步:基础对应关系:**
models.Employee 表示从那张表进行查询,即from.
objects是中间件的名称,不用管
.all表示SELECT *
如果是要取具体的字段,就用values(“字段名”),比如values(“dept”)就相当于 SELECT dept.
可见,如果想要达到select dept,AVG(salary) from employee的效果,我的ORM语句对应的部分是:

```python
models.Employee.objects.values("dept,"平均数的别名")
```

现在平均数的别名还不知道,继续往下看分组:

**第二步:分组与聚合处理**

```
models.Employee.objects.values("dept,"平均数的别名").annotate()
```

这里的annotate()如果不传参数,表示按照.all()全部分组,一般不这样使用..annotate之前的value表示GROUP BY之后的内容,所以`values("dept").annotate(my_avg=Avg("salary"))`就表示按照dept进行分组,然后对salary进行求和,得到分组的结果.

**第三步:取分组后结果**
由于分组结果算出来了,是一个QuerySet,按照我们的需求,最前边是SELECT dept, Avg(salary),现在Avg(salary)的别名有了,所以要在最后再加上.values(“dept”,”my_avg”).
最终结果和原来SQL语句用颜色标识的对应关系是:
`models.Employee.objects.values("dept").annotate(my_avg=Avg("salary").values(dept, "my_avg")`
`select dept,AVG(salary) from employee group by dept;`
ORM语句橙色的部分其实也涵盖了SQL语句中的Avg(salary),这是因为ORM里使用了别名.实际对应关系还是可以很清楚的看出来.

第二个例子是连表查询,dept_id外键关联到dept表.
如果想得到和上边例子一样的结果,在SQL中就是先连表再查询,到了ORM里,其实就是通过外键进行的双下划线查询去跨表:
`select dept.name,AVG(salary) from employee inner join dept on (employee.dept_id=dept.id) group by dept_id`;

```python
models.Dept.objects.annotate(avg=Avg("employee__salary")).values("name", "avg")
```

ORM里从Dept表发起查询并且分组,由于dept_id和dept.name是一一对应的,所以从Dept发起查询后直接接.annotate就可以.后边的对应关系中从dept跨表到employee就相当于连表,然后对salary进行Avg操作,其他的对应关系和上边类似.

## **F查询和Q查询**

F和Q其实就是为了解决以面向对象的方式书写SQL操作的时候,对于语句中的and or 等逻辑控制符不方便书写的问题.

### **F查询**

在之前的查询条件中,使用的都是诸如`.filter(id__gt=3)`这种将字段值与常量进行比较的条件,如果需要对两个字段做比较,显然在Python中不能写成`.filter(id__gt=item_id)`,因为item_id并没有在python程序中定义.这个时候就要使用F查询来替代对应的字段,,只要给F的参数传入字段名称即可.
比如在自行编写的行政物品查询系统中,如果要查询所有入库 大于 出库的单据,就写成:

```python
ret = ret_temp = models.Moves.objects.filter(in_num__gte=F("out_num"))
```

在一张表用自身数据做控制的时候,就要用到F查询,经常在各种内容管理系统中使用,比如列出点赞数高于踩数的条目等等.
除了进行比较之外,F查询在根据自己的字段值更新值的时候比较常用,比如要在出入库表上的出库数字增加10:

```python
models.Moves.objects.update(in_num = F("in_num") + 10)
```

这里要注意的就是update方法,仅可以使用在QuerySet上,单个对象(行数据)没有该方法.该方法返回的是受影响的行数.

### **Q查询**

Q查询是为了解决条件查询的时候方便使用or 等逻辑符的问题,将条件独立出来,然后用逻辑运算符连结来得到最终条件.
默认情况下,filter等条件查询内部可以使用多个条件语句,用逗号分割,这些条件之间是and的关系,比如查询每一张出入库单,要求出库大于入库而且入库大于50,就可以写成:

```python
ret_temp = models.Moves.objects.filter(in_num__gte=F("out_num"), in_num__gt=50)
```

其中的`in_num__gte=F("out_num")`和`in_num__gt=50`是and关系.如果要改成or关系,就要使用Q对象将条件独立出来,然后用&(表示and)和|(表示or)还有~(表示取反)来连结各个Q对象.
如果要查询入库小于出库或者出库大于20的单据:

```python
from django.db.models import F,Q
ret_temp = models.Moves.objects.filter(Q(in_num__lte=F("out_num")) | Q(out_num__gt=20))
```

至此,已经学习完了[DJango 1.11 QuerySet API reference ](https://docs.djangoproject.com/en/1.11/ref/models/querysets/)里的Methods that return new QuerySets,也就是返回查询结果的部分.还有一部分API没有接触到,列在下边:

|                    返回QuerySet的方法补充                    |                                                              |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                            方法名                            |                             解释                             |
|               dates(field, kind, order=’ASC’)                | 针对DateField和DateTimeField类型的字段,取出其中和日期相关的部分,去重,然后排序.返回的是一个QuerySet对象,其中的每一个元素是datetime.date()对象.kind表示精确度,只能取”year”,”month”和”day”三个字符串值.order的ASC表示正向排序,DESC表示逆向排序 |
|    datetimes(field_name, kind, order=’ASC’, tzinfo=None)     | 功能与dates类似.区别是针对DateTimeField类型的字段, kind 只能是 “year”, “month”, “day”, “hour”, “minute” 或 “second”.tiinfo的参数必须是一个 datetime.tzinfo 对象,如果设置为None,Django会使用当前时区. |
|                            none()                            | 不管前边查询条件如何,带上这个方法就一定返回一个空的QuerySet对象 |
|                 union(*other_qs, all=False)                  | 左右连两个查询结果,使用方法是 queryset1.union(queryset2).如果all=True,则允许重复项. |
|                   intersection(*other_qs)                    | 获取多个QuerySet中相同的查询结果.使用方法是qs1.intersection(qs2, qs3) |
|                    difference(*other_qs)                     | 获取只在调用方法的QuerySet中存在,不在参数的QuerySet中存在的结果,使用方法是qs1.difference(qs2, qs3) |
|                   select_related(*fields)                    | 相当于通过外键进行连表查询.使用方法e = Entry.objects.select_related(‘blog’).get(id=5),表示选中id为5的Entry记录通过外键关联的blog里的对象. |
|                  prefetch_related(*lookups)                  | 这个与select类似,主要是性能相关,会预先通过外键进行查找,再次使用查询的时候会到临时表里查询.详细看[这里](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#prefetch-related) |
| extra(select=None, where=None, params=None, tables=None, order_by=None, select_params=None) | 当ORM有时候难以描述一个复杂的WHERE查询的时候,用extra构造额外的查询条件,比如子查询.详细看[这里](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#extra) |
|                        defer(*fields)                        | 参数为字段名,用于在结果集中排出某些列,排出之后,列就没有了对应的字段名的属性 |
|                        only(*fields)                         |                  仅取指定的某些字段的信息.                   |
|                         using(alias)                         | 如果Django中设置了超过一个数据库,在查询的时候用此方法指定使用哪个数据库,alias是数据库的别名,不加则表示使用”default”数据库.数据库的别名是指的settings.py里DATABASES参数的键名. |
|      select_for_update(nowait=False, skip_locked=False)      | 表示所有查询结果在完成事务之前都不能被改动,具体看[这里](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#select-for-update) |
|        raw(raw_query, params=None, translations=None)        | 执行原生的SQL语句,返回一个 django.db.models.query.RawQuerySet对象,可以像QuerySet一样遍历取出各行数据. |

上边的所有方法都是用来查询,也就是返回QuerySet对象,或者是QuerySet封装的字典或者其他对象(比如values, values_list, date等方法),相当于在MySQL中查找的结果.对于查询以外的方法,主要是涉及到增删改查,将在下一部分学习.

# **其他方法 Methods that do not return QuerySets**

所谓其他方法,也就是[不返回QuerySet的方法](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#methods-that-do-not-return-querysets).

|           不返回QuerySet的方法            |                                                              |
| :---------------------------------------: | :----------------------------------------------------------: |
|                   条件                    |                             解释                             |
|               get(**kwargs)               | 参数为关键字的查询条件,返回一个结果,如果找不到,会报 DoesNotExist 异常.如果找到多个,会报 MultipleObjectsReturned 异常.对一个QuerySet对象使用不带参数的get(),可以让其只返回第一个对象.今后遇到查询,还是优先使用filter方法比较好 |
|             create(**kwargs)              | 接受参数来建立一个对象,使用.create()方法就包含了.save()的过程,详细看[这里](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#create) |
|  get_or_create(defaults=None, **kwargs)   | 用给定的查询条件寻找对象,如果找不到,就建立一个.这个方法返回一个元组(object,created),元组第一个元素是查询结果或者新建的数据,第二个是一个布尔值,如果新建了就是True,否则是False.这个方法实际过程相当于先去调用get(),如果产生了DoesNotExist异常,则会新建一个同样参数的对象.注意关键字参数里名称叫做defaults的关键字是不会成为内含的get()调用中的参数,只会在新建的时候发生作用.具体看[这里](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#get-or-create) |
| update_or_create(defaults=None, **kwargs) | 和get_or_create类似,如果字段存在则更新,否则创建.defautls也是表示创建时候的其他字段. |
|    bulk_create(objs, batch_size=None)     | 批量插入(新增)数据,objs是一个列表,batch_size表示一次插入的个数.具体看[这里](https://docs.djangoproject.com/en/1.11/ref/models/querysets/#bulk-create) |
|           in_bulk(id_list=None)           | id_list是一个主键列表,根据其中的主键,返回一个字典,键是主键,值是对应的数据行对象.如果不传参数,则返回所有主键和对应数据行构成的字典.在按照主键查询的时候经常用此方法. |
|                iterator()                 |            QuerySet的方法,使用之后变成一个迭代器.            |
|          latest(field_name=None)          | 需要指定字段名,字段必须是时间类型的字段,直接返回时间最晚的那一行记录. |
|         earliest(field_name=None)         | 和latest用法相同,返回时间最早的那一条记录,和latest经常使用在一些内容管理的查找上 |
|             update(**kwargs)              | 用在QuerySet上,更新QuerySet里所有对象的字段,隐含了save操作,返回受影响的行数.update不能够用在关联关系的操作上,不能直接通过外键名__字段名去更新其他表.之前说过查询的时候多用filter,因此今后就用update与filter搭配使用. |
|                 delete()                  | 用在单个对象或者QuerySet上,表示将选择的行从表里删除,返回受影响的行数. |
|               as_manager()                | 在每次写查询的`models.Book.objects`中的objects就是一个对象管理器,对象管理器的默认名称就是objects.这个方法就是产生一个带有QuerySet所有方法的管理器,替代了objects这个名字,具体看[这里](https://docs.djangoproject.com/en/1.11/topics/db/managers/#manager-names) |

# **补充部分**

### **查看实际生成的SQL语句**

在使用ORM的时候,有的时候需要知道实际生成的SQL语句,将如下部分复制到settings.py中,设置DEBUG=True,每次执行ORM操作的时候,就可以在python console中看到实际的SQL语句.

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console':{
            'level':'DEBUG',
            'class':'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'propagate': True,
            'level':'DEBUG',
        },
    }
}
```

### **在python程序中调用Django环境**

一般来说我们在启动Django程序的时候才运行在Django环境中,单独写一个py文件,导入Django的库是无法进行操作的.
实际上Django环境是写在Django_admin等内容中的,直接使用会提示导入环境配置文件,只要根据提示操作即可,这里提供直接可供粘贴的代码:

```python
import os

if __name__ == '__main__':
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "BMS.settings")
    import django
    django.setup()
#  在以上语句结束之后,就可以像编写views,models等文件中来导入Django的各种模块来进行操作,结果也与像在实际的Django环境中得到的结果一样
    from app01 import models
    books = models.Book.objects.all()
    print(books)
```