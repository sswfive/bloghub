---
title: django中ORM字段总结
date: 2020-10-04 22:07:49
tags: 
  - Python
  - Django
---
# django中ORM字段总结


##  **表与字段**

对于ORM来说,表与字段就是类与类属性,是ORM模型的基础,没有类和类属性,就根本无法使用数据库,同时在Django里设置类和类属性,对于实际的MySQL中的表有何影响,也是必须要掌握的内容.

## **类=表**

在ORM中,对应一张数据表的,是一个[类](https://docs.djangoproject.com/en/1.11/topics/db/models/)(一般称为一个Model,一个模型),这个类必须具有如下特点:

1. 继承`django.db.models.Model`类,每一个类属性表示一个数据表的字段(database field)
2. 某个app要使用的类,需要写在该app目录下的models.py文件内部,还要在settings.py文件的INSTALLED_APPS内注册该APP
3. 满足上边两个条件后,Django为这个类自动提供了一系列的API可供视图函数使用,也就是各种类方法,主要用于增删改查数据.

表还有一些其他的内容,比如META类,一些默认类方法,还有表之间的继承关系,等到后边再学习,先来看最核心的内容-表的字段.

## **属性=字段**

表的内容相对简单,就是建立一个类.而真正对表有影响的,是其中的[字段(Field)](https://docs.djangoproject.com/en/1.11/ref/models/fields/),非常重要,因为字段不仅代表这表的实际内容,在ORM模型下,字段的设置还会对MySQL里实际建立的表产生非常大的影响.
在之前的项目中,使用过的字段有AutoField,CharField,ForeignKey,ManyToManyField.这些对于开发程序来说远远不够,现在要来详细的看一下所有的字段及对应关系.

在Django中,所有的字段(Field),都是由两部分组成的,即[字段类型(Field Types)](https://docs.djangoproject.com/en/1.11/ref/models/fields/#field-types)和[字段选项(Field Options)](https://docs.djangoproject.com/en/1.11/ref/models/fields/#field-options)构成.例如我们之前写的:

```python
id = models.AutoField(primary_key=True)
```

其中`models.AutoField`叫做字段类型;`primary_key`叫做字段选项或者说字段参数,字段选项在设置的时候通过类似关键字参数的方式设置.下边分开介绍这两部分.

### **字段类型 Field types**

|          字段类型          |                                                              |
| :------------------------: | :----------------------------------------------------------: |
|          属性名称          |                             解释                             |
|         AutoField          | 自增字段,是一个IntegerField(32位INT).如果手动设置了AutoField,必须指定字段选项primary_key=True. 如果一个表里边没有任何AutoField,所有的字段也都没有被设置primary_key=True,则会自动创建一个列名叫做id,为IntegerField的自增主键列. |
|        BigAutoField        |                   是一个64位INT的AutoField                   |
|      BigIntegerField       | 64位INT的 IntegerField,范围 -9223372036854775808 到 9223372036854775807. |
|        BinaryField         | 存放原始二进制数据,功能有限,较少使用,在Model Form里也没有对应表单. |
|        BooleanField        | 布尔值.布尔字段的默认值是None,除非自定义 Field.default.默认布尔字段不能为null,如果要为null,需要使用 NullBooleanField |
|         CharField          | 相当于MySQL里的Vachar,必须指定max_length值.对应的表单元素是input-text.如果要存储大量文字,用TextField字段.ORM中没有对应MySQL中的char类型,如果需要可以自定义 |
|         DateField          | 时间字段,很重要.表示一个日期,对应的Ptyhon中的对象是 datetime.date实例.有两个选项auto_now=True表示每次该行数据被.save()修改的时候记录当前时间,用于记录每次修改的时间,默认传进来的数据来自 datetime.date.today(). auto_now_add=True表示在生成该对象的时候记录一次当前时间,之后不再改变,用于记录生成时间戳,默认传进来的数据是 django.utils.timezone.now(). |
|       DateTimeField        | 这个对应的是 datetime.datetime,即带有时间和日期的对象.其他设置和DateField一样. |
|        DecimalField        | 固定精确度的小数,对应python的[Decimal类](https://docs.python.org/3/library/decimal.html#decimal.Decimal)有两个选项:max_digits表示总长度,decimal_places表示小数点后的长度. |
|         EmailField         | 内部是一个CharField,默认长度为254,Django Admin和Model Form会自动检查Email地址的有效性. |
|       DateTimeField        | 这个对应的是 datetime.datetime,即带有时间和日期的对象.其他设置和DateField一样. |
|         FloatField         |                       存放双精度浮点数                       |
|        IntegerField        |                  32位INT,标准的整数类型字段                  |
|     SmallIntegerField      |                 小整数类型,从-32768 到 32767                 |
|        IntegerField        |                  32位INT,标准的整数类型字段                  |
|    PositiveIntegerField    |                32位正整数类型,0 to 2147483647                |
| PositiveSmallIntegerField  |                   小正整数类型, 0 to 32767                   |
|         TextField          |                          存放长字符                          |
|   GenericIPAddressField    | 内部是CharField,提供验证IPV4和IPV6.两个选项,protocol用于指定协议是IPV4还是IPV6.unpack_ipv4可以提供解析,需要protocol设置为”both” |
|          URLField          |    内部是CharField,Django Admin和Model Form中提供验证URL     |
|         SlugField          | 内部是CharField,Django Admin和Model Form中提供验证,只能为字母,数字,下划线和连接符(减号) |
| CommaSeparatedIntegerField |                 逗号分隔的数字,为字符串类型                  |
|         UUIDField          | 字符串类型,用于提供对UUID的验证,对应Python中的[UUID模块](https://docs.python.org/3/library/uuid.html#uuid.UUID),Django Admin和Model Form中提供UUID的验证 |
|       FilePathField        | 用于存放文件路径,Django Admin 和Model Form中提供读取文件夹下文件的功能,这里可以存放一个文件,也可以存放一个文件的路径.详细看[这里](https://docs.djangoproject.com/en/1.11/ref/models/fields/#filepathfield) |
|         FileField          | 字符串,用于接受文件,选项upload_to表示上传文件的保存路径,storage=None,是选择存储组件,具体看[这里](https://docs.djangoproject.com/en/1.11/ref/models/fields/#filefield) |
|         ImageField         |       和FileField一样,只不过验证上传的文件一定是图像.        |
|         TimeField          |     对应datetime.time实例,其他用法和另外两个时间模块一样     |
|       DurationField        | 长整数类型,用于存放时间间隔,ORM中获取的值是datetime.timedelta类型 |

除了上边常用的内置字段之外,还可以自定义字段,在models.py里新建一个类继承models.Field类,具体以后先研究,先把示例代码贴在这里:

```python
class FixedCharField(models.Field):
    """
    自定义的char类型的字段类
    """
    def __init__(self, max_length, *args, **kwargs):
        self.max_length = max_length
        super(FixedCharField, self).__init__(max_length=max_length, *args, **kwargs)

    def db_type(self, connection):
        """
        限定生成数据库表的字段类型为char，长度为max_length指定的值
        """
        return 'char(%s)' % self.max_length


class Class(models.Model):
    id = models.AutoField(primary_key=True)
    title = models.CharField(max_length=25)
    # 使用自定义的char类型的字段
    cname = FixedCharField(max_length=25)
```

### **字段选项 Field options**

下边的选项可以用于所有的字段:

| 字段选项(字段参数) |                                                              |
| :----------------: | :----------------------------------------------------------: |
|      选项名称      |                             解释                             |
|        null        | 表示字段是否可以为空,默认为False即不能为空.如果设置为True,不传值的时候Django会存入MySQL的Null值.对于基于字符串的字段如果设置为True,意味着可能存在Null和空字符串两种形式.一般最好验证一下字符串不为空. |
|       blank        | 是否可以为空值,注意空值和Null是不同的,Null是纯粹对于数据库来说的属性.如果设置为True,表单可以提交空白的数据,否则就不行.对于某些字符串,最好同时设置null=False和blank=False |
|      choices       | 用于页面上的选择框标签，需要先提供一个二维的二元元组，第一个元素表示存在数据库内真实的值，第二个表示页面上显示的具体内容。在浏览器页面上将显示第二个元素的值。 |
|     db_column      | 该参数用于定义当前字段在数据表内的列名。如果未指定，Django将使用变量名作为列名. |
|      db_index      |       这个参数接受布尔值,为True表示为当前字段创建索引        |
|      default       | 字段的默认值，可以是值或者一个可调用对象。如果是可调用对象，那么每次创建新对象时都会调用。设置的默认值不能是一个可变对象，比如列表、集合等等。lambda匿名函数也不可用于default的调用对象，因为匿名函数不能被migrations序列化。 |
|      editable      | 如果设为False，那么当前字段将不会在admin后台或者其它的ModelForm表单中显示，同时还会被模型验证功能跳过。参数默认值为True。 |
|   error_messages   | 用于自定义错误信息。参数接收字典类型的值。字典的键可以是null、 blank、 invalid、 invalid_choice、 unique和unique_for_date其中的一个。 |
|     help_text      | 额外显示在表单部件上的帮助文本。使用时请注意转义为纯文本，防止脚本攻击。 |
|    primary_key     | 为某个字段设置了primary_key=True，则当前字段变为主键，并关闭Django自动生成id主键的功能。primary_key=True隐含null=False和unique=True的意思。一个模型中只能有一个主键字段！ |
|       unique       | 设为True时，在整个数据表内该字段的数据不可重复。当unique=True时，db_index参数无须设置，因为unqiue隐含了索引。对于ManyToManyField和OneToOneField关系类型，该参数无效。 |
|    verbose_name    | 为字段设置一个人类可读，更加直观的别名。对于每一个字段类型，除了ForeignKey、ManyToManyField和OneToOneField这三个特殊的关系类型，其第一可选位置参数都是verbose_name。如果没指定这个参数，Django会利用字段的属性名自动创建它，并将下划线转换为空格。 |
|     validators     |                 运行在该字段上的验证器的列表                 |

## **关系型字段**

上边介绍了表的所有数据字段以及常用的字段选项.在ORM中,还有一类重要的字段就是关系型字段.在MySQL中,仅仅只有外键中一种类型,实现多对多需要自行写一张表来对应.而在ORM模型中,抽象出了一对多,多对多关系并封装好,体现在了关系型字段的设置上.关系型的字段按照关系来学习:

### **ForeignKey 一对多关系**

外键类型在ORM中用来表示外键关联关系，一般把ForeignKey字段设置在 ‘一对多’中’多’的一方。
ForeignKey可以和其他表做关联关系同时也可以和自身做关联关系。设置ForeignKey的格式如下:

```python
theclass = models.ForeignKey(to="Classes",to_field="name" related_name="students",on_delete=models.CASCADE)
```

ForeignKey的几个字段参数介绍如下:
**to=**
其中的to=表示外键关联到哪个表,可以直接为类名,如果类还没建立,可以用字符串的类名.to_field表示关联到哪个字段,如果不设置,Django默认会关联到主键.

**related_name**
related_name表示反向操作的时候代替的表名,比如学生表关联到班级表:

```python
class Classes(models.Model):
    name = models.CharField(max_length=32)

class Student(models.Model):
    name = models.CharField(max_length=32)
    theclass = models.ForeignKey(to="Classes")
```

这里外键设置在学生里,如果要查一个班级对应的所有学生,操作的主体是Classes对象,写成:

```
models.Classes.objects.first().student_set.all()
```

这里的_set是固定写法,即去反查设置了外键的那一张表.student是自动小写的类名.
如果Student表里的theclass外键设置了related_name如下:

```python
class Student(models.Model):
    name = models.CharField(max_length=32)
    theclass = models.ForeignKey(to="Classes", related_name="students")
```

则上边通过班级表查对应学生可以写成:

```python
models.Classes.objects.first().students.all()
```

**related_query_name**
反向查询操作时，使用的连接前缀，用于替换表名。即在反向操作的时候给表起个别名.related_name是给外键字段起一个别名.

**on_delete**
这个表示当删除字段的时候,对应的行为,有如下几种.

|   on_delete参数    |                                                              |
| :----------------: | :----------------------------------------------------------: |
|       参数值       |                             解释                             |
|   models.CASCADE   |      删除关联数据，与之关联也删除.一般大型数据库里不用       |
| models.DO_NOTHING  | 什么都不做,如果数据库强制检查操作的完整性,有可能引发 IntegrityError 错误 |
|   models.PROTECT   |             引发错误ProtectedError,不会删除数据              |
|  models.SET_NULL   |      删除关联数据，将外键设置为空,需要外键可以设置为空       |
| models.SET_DEFAULT |          将外键设置为默认值,前提是外键设置过默认值           |
|     models.SET     | 删除关联数据， a. 与之关联的值设置为指定值，设置：models.SET(值) b. 与之关联的值设置为可执行对象的返回值，设置：models.SET(可执行对象) |

**db_constraint**
表示是否建立约束,默认是True,但一般大型数据库是建立软约束,即建立外键但是不设置约束,每次通过程序来检查数据一致性.设置成False的话依据ORM关系字段的查询将无法生效.

### **OneToOneField 一对一关系**

一对一关系经常是用来扩展存储的内容,实际上用一张表也能装下,主要用于对于某些字段查询特别频繁的情况,将一张表中的字段单独分离出一个表.
一对一关系和unique=True的外键功能是相同的,只不过在反查的时候,不是返回一组对象,而是直接返回单个对象(因为是一对一)
字段选项和外键都相同.

### **ManyToManyField 多对多关系**

在之前已经使用过一次多对多关系,其写法与外键也类似,但是看到结果是在MySQL里自动生成了用于多对多关系的表.现在看一看详细的控制方法
字段参数:

|   多对多字段参数   |                                                              |
| :----------------: | :----------------------------------------------------------: |
|       参数值       |                             解释                             |
|         to         |         设置要关联的表,如果设置成”self”表示关联自身          |
|    related_name    |              和外键一样,给这个多对多字段起别名               |
| related_query_name |                 和外键一样,给查询表名起别名                  |
|    symmetrical     | 仅用于多对多关联自身,指定内部是否创建反向操作的字段,默认为True,即可以用_set反向查询 |
|      through       | 无该参数,Django自动创建多对多关系表.该参数是手动创建多对多表的名称. |
|   through_fields   |                       设置关联的字段。                       |
|      db_table      |            默认创建第三张表时，数据库中表的名称。            |

但看字段参数还有点不明白,在MySQL里多对多关系是通过创建一张表格来对应的.要了解ManyToMany,就要了解ORM模型里创建多对多关系的三种方法:

第一种方法,自行创建第三张表,不使用ManyToMany:

```python
class Book(models.Model):
    title = models.CharField(max_length=32, verbose_name="书名")


class Author(models.Model):
    name = models.CharField(max_length=32, verbose_name="作者姓名")


# 自己创建第三张表，分别通过外键关联书和作者
class Author2Book(models.Model):
    author = models.ForeignKey(to="Author")
    book = models.ForeignKey(to="Book")

    class Meta:
        unique_together = ("author", "book")
```

这种方法自然无法通过ManyToMany字段来直接查询取值,想要查询关系就必须通过多对多表拿到结果再过滤.

第二种方法,通过ManyToManyField自动创建多对多关系表:

```python
class Book(models.Model):
    title = models.CharField(max_length=32, verbose_name="书名")


# 通过ORM自带的ManyToManyField自动创建第三张表
class Author(models.Model):
    name = models.CharField(max_length=32, verbose_name="作者姓名")
    books = models.ManyToManyField(to="Book", related_name="authors")
```

这种情况下想要查询作者对应的书,是可以使用Django提供的API直接查询的,就像之前项目里写的`author_obj.book.set(books)`直接在多对多关系里设置了对应关系.

第三种方法,设置ManyTomanyField并指定自行创建的第三张表:

```python
class Book(models.Model):
    title = models.CharField(max_length=32, verbose_name="书名")


# 自己创建第三张表，并通过ManyToManyField指定关联
class Author(models.Model):
    name = models.CharField(max_length=32, verbose_name="作者姓名")
    books = models.ManyToManyField(to="Book", through="Author2Book", through_fields=("author", "book"))
    # through_fields接受一个2元组（'field1'，'field2'）：
    # 其中field1是定义ManyToManyField的模型(ManyToMany字段所在的表)外键的名（author），field2是关联目标模型 （book）的外键名。


class Author2Book(models.Model):
    author = models.ForeignKey(to="Author")
    book = models.ForeignKey(to="Book")

    class Meta:
        unique_together = ("author", "book")
```

虽然第三种方式也使用了ManyToMany字段,但这种方式与第一种类似,无法使用set、add、remove、clear等内建API来管理多对多的关系了，需要通过第三张表自己取查询结果再操作.

到这里基本上学习完了定义表时候的操作,包括类和类属性,关联关系的各种设置.
此外,还要对表再补充一点,就是类内可以再定义一个叫[META的类](https://docs.djangoproject.com/en/1.11/ref/models/options/),保存着这个表的一些其他属性.
建立META类很简单,只要在表的类内定义一个类如下:

```python
class Meta:
    unique_together = ("author", "book")
```

META类的类属性都有着特殊的意义:

| META类的类属性  |                                                              |
| :-------------: | :----------------------------------------------------------: |
|     属性名      |                             解释                             |
|    abstract     |   设置为True表示这是一个ABC类,这个类的字段会应用到子类中.    |
|    app_label    | 类对应的app名称.如果当前这个类是在app之外定义的,将app_label设置为具体的app名称可以指定这个类是为哪个app准备的 |
|    db_table     |       指定表名,不指定则Django自动按照app_类名生成表名        |
|     indexes     | 指定编制索引的字段和索引名称,具体看[这里](https://docs.djangoproject.com/en/1.11/ref/models/options/#indexes) |
| unique_together | 参数为字段元组,表示哪一组字段进行联合唯一约束,具体看[这里](https://docs.djangoproject.com/en/1.11/ref/models/options/#unique-together) |
| index_together  |               参数为字段名的列表,表示联合索引                |
|  verbose_name   |                    给这个表设置一个别名.                     |
|    ordering     | 参数为一个字段名称的列表,表示默认按照哪一个字段排序.设置了该属性之后,查询结果集才可以被reverse(). |

可见ORM对于表的处理,还是进行了尽可能详细的区分,表的基本属性,排序和索引字段,以及具体字段的设置都有对应的方法.今后在设置数据库的时候,尽量使用各种特性.