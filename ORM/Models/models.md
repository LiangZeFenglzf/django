---
Model
---





[TOC]

# Model

1. 模型父类 是[`django.db.models.Model`](https://docs.djangoproject.com/en/4.1/ref/models/instances/#django.db.models.Model)
2. model属性 代表 数据库表字段
3. model 映射 特定数据库表



# Fields

[字段Option;types;relationship fields]: https://docs.djangoproject.com/en/4.1/ref/models/fields/#model-field-types

Model  == the list of database fields

Field specified by model class attributes

## field option

Each field takes a certain set of field-specific arguments (documented in the [model field reference](https://docs.djangoproject.com/en/4.1/ref/models/fields/#model-field-types)).

字段的额外 特性。

特别的 choices 有 get_FOO_display()这个api。

## PK

id是默认主键，Django默认添加到model。

通过primary_key可 特定某字段为主键

[]: https://docs.djangoproject.com/en/4.1/topics/db/models/#automatic-primary-key-fields

## Verbose 

指定字段备注名。 默认是 字段 小写 ，下划线转空格。

对于关联字段，关联模型是第一个argument;

对于非关联字段，如果需要指定备注名，verbose_name需要是第一个argument

有啥用?

## Relationships

[关联字段]: https://docs.djangoproject.com/en/4.1/topics/db/models/#automatic-primary-key-fields

### mang to many

[`ManyToManyField`](https://docs.djangoproject.com/en/4.1/ref/models/fields/#django.db.models.ManyToManyField) requires a positional argument: the class to which the model is related.

提供一个指名关联模型的 位置参数就行

### many to one

[`django.db.models.ForeignKey`](https://docs.djangoproject.com/en/4.1/ref/models/fields/#django.db.models.ForeignKey)

把ForeignKey当做一个FieldType，当作模型class的一个attribute

提供位置参数 指定哪个模型class关联上。

建议这个外键 属性名 可以是关联实体名小写，当然randomly

#### field option define how the relationship work

[ForeigmnKey字段选项]: https://docs.djangoproject.com/en/4.1/ref/models/fields/#foreign-key-arguments

##### on_delete

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

host_product 关联多个host

通过host_product_id关联，

如果在host_product表里面 host_product_id不是唯一，u也就是存在重复的host_product记录

后续CloudOsHost.objects.all().values('hot表字段''host_product字段')时

会因为这个host_product_id重复  出现  host_id重复。

场景  返回host_list  ,host_list既有host本身信息host_id 也要查出host_product_id,host_product_name;2）host 定义foreignkey 引用Host_product  many to one,one是host，many是Host_prouduct；

host_product存的要是有重复host_product那就会导致返回的host_list  host_id也会重复。

#### backwards-related反向获取关联实体

#### example many to one

[去关联一个未定义的模型类，使用模型名充当位置参数，而不是实际的模型classs]: https://docs.djangoproject.com/en/4.1/ref/models/fields/#lazy-relationships



### one to one 