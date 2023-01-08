---
Model
---





[TOC]

# Model

1. 模型父类 是[`django.db.models.Model`](https://docs.djangoproject.com/en/4.1/ref/models/instances/#django.db.models.Model)
2. model属性 代表 数据库表字段
3. model 映射 特定数据库表

本文档https://docs.djangoproject.com/en/4.1/topics/db/models/#

引用文档:https://docs.djangoproject.com/en/4.1/ref/models/fields/#

# 1Fields

[字段Option;types;relationship fields]: https://docs.djangoproject.com/en/4.1/ref/models/fields/#model-field-types

Model  == the list of database fields

Field specified by model class attributes

## 1.1field option

Each field takes a certain set of field-specific arguments (documented in the [model field reference](https://docs.djangoproject.com/en/4.1/ref/models/fields/#model-field-types)).

字段的额外 特性。

特别的 choices 有 get_FOO_display()这个api。

## 1.2PK

id是默认主键，Django默认添加到model。

通过primary_key可 特定某字段为主键

[]: https://docs.djangoproject.com/en/4.1/topics/db/models/#automatic-primary-key-fields

## 1.3Verbose 

指定字段备注名。 默认是 字段 小写 ，下划线转空格。

对于关联字段，关联模型是第一个argument;

对于非关联字段，如果需要指定备注名，verbose_name需要是第一个argument

有啥用?

## 1.4Relationships

[关联字段]: https://docs.djangoproject.com/en/4.1/ref/models/fields/#module-django.db.models.fields.related

对于一下三种关联字段声明，统一by including it as a class attribute of your model

### 1.4.1.mang to many

[`ManyToManyField`](https://docs.djangoproject.com/en/4.1/ref/models/fields/#django.db.models.ManyToManyField) requires a positional argument: the class to which the model is related.

提供一个指名关联模型的 位置参数就行

[related_objects]: https://docs.djangoproject.com/en/4.1/ref/models/relations/	"里面clear能取消关联但不删除关联实体，但是要通过RelatedManager才能使用clear method"

clear方法 取消关联

#### 1.4.1.2Extra fields on ManyToMany

django在你声明 ManyToMany字段后会自动偷偷建立一个关联表；

问题是这个关联表 能表达 关联关系有限；

未主动声明中介表，Django自动生成的关联表只有3个字段

[id,sourc_model_id,other_model_id]: https://docs.djangoproject.com/en/4.1/ref/models/fields/#django.db.models.ManyToManyField.through

你可以手动声明一个关联表 作为中介 ；在这个中介表 添加一些字段表明  关联原因，关联目的，关联时间等等额外信息。

#### 1.4.1.3 manyToManyFieldOption

[option]: https://docs.djangoproject.com/en/4.1/ref/models/fields/#manytomanyfield

对于多对多关联实体可以通过定义的related_manager 方法得到；

related_manager的名字就是related_name，同ForeignKey定义一致。

[through¶](https://docs.djangoproject.com/en/4.1/ref/models/fields/#django.db.models.ManyToManyField.through)

有了中介表 在声明ManyToManyField时 添加argument位置参数 field_option：through，through='关联表名'/关联实体类，用名字是为了还没有创建表。

[through_fields¶](https://docs.djangoproject.com/en/4.1/ref/models/fields/#django.db.models.ManyToManyField.through_fields)

`through_fields` accepts a 2-tuple `('field1', 'field2')`, where `field1` is the name of the foreign key to the model the [`ManyToManyField`](https://docs.djangoproject.com/en/4.1/ref/models/fields/#django.db.models.ManyToManyField) is defined on (`group` in this case), and `field2` the name of the foreign key to the target model (`person` in this case).

也就是说 through_fileds这个option是以二元元组出现 (field1,field2) field1是 定义了ManyToManyField的实体，定义在关联表的字段名。field2是ManyToManyField去取的实体，定义在关联实体表的字段名。

至于为什么要在声明ManyToMany的时候指定through_fields这个option那是因为....上面那个链接提到了 中介表 还有额外字段也指向了TargetModel，避免歧义产生。

#### 1.4.1.4多对多关联实体的关联取消发生什么？

```
clear()
>>> # Note that this deletes the intermediate model instances
```

##### 疑惑？没有使用related_name

```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):
        return self.name

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

    def __str__(self):
        return self.name

class Membership(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)
```

为什么这里直接用定义的关联字段名members 而没有使用related_name .还是members的related_name默认名字是字段名

##### 解答：参考1.4.2.3backwards

[解答出处]: https://docs.djangoproject.com/en/4.1/ref/models/relations/#django.db.models.fields.related.RelatedManager.clear

对定义了ManyToManyField的SOurceModel,使用SouceModelName.关联MTM字段名.clear;

对于没有定义ManyToManyField的TargetModel，使用related_name

TargetModelName.FOO_set.clear()；Foo是SourceModel的小写名。

和多对一关联来讲，多对多这种查询反向不是很严格。毕竟多对多谁也不承认自己地位低是反向查。

### 1.4.2.many to one

[文档位置]: https://docs.djangoproject.com/en/4.1/topics/db/models/#many-to-one-relationships

[`django.db.models.ForeignKey`](https://docs.djangoproject.com/en/4.1/ref/models/fields/#django.db.models.ForeignKey)

把ForeignKey当做一个FieldType，当作模型class的一个attribute

提供位置参数 指定哪个模型class关联上。

建议这个外键 属性名 可以是关联实体名小写，当然randomly

#### See also

##### 1外键数据库表现:列名 ==属性名_id

[关联字段数据库列名]: https://docs.djangoproject.com/en/4.1/ref/models/fields/#database-representation	"默认：字段名_id，也可通过option:db_column指定"

##### 2field option define how the relationship work

[ForeigmnKey字段选项]: https://docs.djangoproject.com/en/4.1/ref/models/fields/#foreign-key-arguments

###### on_delete

###### SET_NULL

通过ForeignKey引用的实体如果被删除，如果设置on_delete这个option为set_null 那么引用的实体就会是Null,当然使用set_null前提 ForeignKey需要满足Null=True；

###### DO_NOTHING

啥也不干；如果数据库设定了 引用完整性，那么引用实体删除的时候会触发[`IntegrityError`](https://docs.djangoproject.com/en/4.1/ref/exceptions/#django.db.IntegrityError)

###### related_name

many_to_one  related_name定义的是one的名字。

```en
The name to use for the relation from the related object back to this one
定义的是ONE的名字。
```

同时也是[`related_query_name`](https://docs.djangoproject.com/en/4.1/ref/models/fields/#django.db.models.ForeignKey.related_query_name) 的默认值

related_query_name是为了实现反向filter()查询，什么是反向

通过One去查many叫反向；某部门

的所有产品

many查One是正向

###### to_field

django默认使用主键去表join【主键一定是全表唯一的】。如果你想指定别的字段，要保证字段unique=True

不保证这个 就会出现查出来 主表1 副表2

最后查出来objects.all()里面 host_id 同为***的有两条

```
host_product 关联多个host

通过host_product_id关联，

如果在host_product表里面 host_product_id不是唯一，u也就是存在重复的host_product记录

后续CloudOsHost.objects.all().values('hot表字段''host_product字段')时

会因为这个host_product_id重复  出现  host_id重复。

场景  返回host_list  ,host_list既有host本身信息host_id 也要查出host_product_id,host_product_name;2）host 定义foreignkey 引用Host_product  many to one,one是host，many是Host_prouduct；

host_product存的要是有重复host_product那就会导致返回的host_list  host_id也会重复。
```



##### 1.4.2.3backwards-related反向获取关联实体

[]: https://docs.djangoproject.com/en/4.1/topics/db/queries/#following-relationships-backward

多对一ManyToOne

Many中  定义  ForeignKey 引用One

One中有一个manger,name是FOO_set FOO是代表Many模型类小写名。

那么One.FOO_set.all()能查出One手下所有的Many;

当然FOO_set能修改，在ForeignKey定义 是添加argument,也就field_option:related_name

那么就可以通过One.自定义的related_name.all()取出 One手下所有的Many

One.自定义related_name是 一个返回Queryset的Manager

###### 3.Q如果related_name就是反向，那么related_query_name是什么》？

related_name是  manager

related_query_name是 用在filter（）语句充当查询条件field__look_up，都是反向查询，用处不一样而已。

 

###### 3.ForeignKeyMethod

多对一关系才有add,create,remove,clear,set方法

##### 4example many to one

[去关联一个未定义的模型类，使用模型名充当位置参数，而不是实际的模型classs]: https://docs.djangoproject.com/en/4.1/ref/models/fields/#lazy-relationships



### 1.4.3.one to one 

This is most useful on the primary key of an object when that object “extends” another object in some way.

当Object extends(继承) 另一个对象，定义一对一字段在，另一个对象的主键上是关键的。

For example, if you were building a database of «places», you would build pretty standard stuff such as address, phone number, etc. in the database. Then, if you wanted to build a database of restaurants on top of the places, instead of repeating yourself and replicating those fields in the `Restaurant` model, you could make `Restaurant` have a [`OneToOneField`](https://docs.djangoproject.com/es/4.1/ref/models/fields/#django.db.models.OneToOneField) to `Place` (because a restaurant «is a» place; in fact, to handle this you’d typically use [inheritance](https://docs.djangoproject.com/es/4.1/topics/db/models/#model-inheritance), which involves an implicit one-to-one relation).

大意就是  餐厅   地点  2个模型类，餐厅继承地点模型类，1） 餐厅模型类定义个地方 one to one字段 因为 餐厅 <<is a>> 地点 2）通过继承，继承是暗含一对一关联的。

#### 1.4.3.1above不用关注

[`OneToOneField`](https://docs.djangoproject.com/es/4.1/ref/models/fields/#django.db.models.OneToOneField) classes used to automatically become the primary key on a model. This is no longer true (although you can manually pass in the [`primary_key`](https://docs.djangoproject.com/es/4.1/ref/models/fields/#django.db.models.Field.primary_key) argument if you like). Thus, it’s now possible to have multiple fields of type [`OneToOneField`](https://docs.djangoproject.com/es/4.1/ref/models/fields/#django.db.models.OneToOneField) on a single model.

[`OneToOneField`](https://docs.djangoproject.com/zh-hans/4.1/ref/models/fields/#django.db.models.OneToOneField) 类通常自动的成为模型的主键，这条规则现在不再使用了（然而你可以手动指定 [`primary_key`](https://docs.djangoproject.com/zh-hans/4.1/ref/models/fields/#django.db.models.Field.primary_key) 参数）。因此，现在可以在单个模型当中指定多个 [`OneToOneField`](https://docs.djangoproject.com/zh-hans/4.1/ref/models/fields/#django.db.models.OneToOneField) 字段。

为什么自动成为主键呢 因为 一对一 餐厅定义一个place one to one 字段，place成为餐厅主键是因为  一对一  一个餐厅一个地点，地点主键值就能查出唯一一条餐厅，一条餐厅查出唯一一条地点。

is a/has a 是java的面向对象  继承概念

[example]: https://docs.djangoproject.com/en/4.1/topics/db/examples/one_to_one/	"hasattr（）"

### 1.6 字段名限制



# 5model inheritance

[doc]: https://docs.djangoproject.com/es/4.1/topics/db/models/#model-inheritance



3种继承风格

## 5.1abstract base class

## 5.2multi_tale inheritance继承已有模型类

继承关系会引入关联 在 子类实体 和 每一个 父类实体，（通过偷偷自动创建OneToOneField，不用显示声明一对一字段)

餐厅  继承  地点

娱乐场所  继承  地点

当地点 是餐厅 能取出餐厅的特有属性；如地点是娱乐场所，地点.餐厅 就会raise exception

手动修改这个关联关系的话可以override:parent_link=True,primary_key=True

3.1提到 一对一字段不必是子模型主键那么只需要parent_link=True

[parent_link]: https://docs.djangoproject.com/es/4.1/ref/models/fields/#django.db.models.OneToOneField.parent_link	"parent_link使用来定义拿到父类模型，不像subclass继承一对一字段是自动偷偷创建的"

那么host_shard当作子类模型 host当作 父类模型

然后查host顺便通过shard查出shard信息。目前是因为host信息一定有{父：基础信息}，shard信息不一定有(子:额外信息)

### 5.2.Q通过字典形式查出来？双下划线？

之前提到是隐式创建了一对一关联字段，那么查的时候values 那也应该 有一个 字段 是 小写 父类模型名 values('lower(父类模型名)__父类属性')

### 5.2.1meta  父类对子类影响如何消除

[]: https://docs.djangoproject.com/es/4.1/ref/models/fields/#django.db.models.OneToOneField.parent_link

ordeIng  置为[]消除父类定义好的排序

## 5.2.2 子类表 再定义一个 关联父类字段须指定related_name

[]: https://docs.djangoproject.com/zh-hans/4.1/topics/db/models/#inheritance-and-reverse-relations

supplier是 subclass继承 place，如果还需要定义属性 ForeignKey/manyTOMany关联place，属性Option需要指定related_name

### 5.2.3Specifying the parent link field

[手动设定父类关联字段parent_link=True]: https://docs.djangoproject.com/zh-hans/4.1/topics/db/models/#specifying-the-parent-link-field

