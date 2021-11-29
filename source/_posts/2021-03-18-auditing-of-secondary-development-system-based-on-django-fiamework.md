---
title: "记一次基于django二次开发系统业务源代码审计"
date:   2021-03-18  14:30:00 +0800
layout: post
tag:
- Code
categories:
- Security
- Code Audit
---

记一次基于django二次开发系统业务源代码审计

-------

## 简述
突然被领导拉去协助产研的同学做代码审计，审计的系统是一个后端业务系统，系统基于django框架做的二次开发，我只负责审计业务代码即可。审计过程中遇到一个文件上传点未按照编码规范，使用拼接的形式查询数据而导致的sql注入漏洞，听起来很魔幻，又因为在审计过程中对于postgresql语法生疏，一定要记录一下，否则后续还得花费时间学习。

## django 如何防止sql注入

> 防御 SQL 注入
>
> SQL 注入是一种让恶意用户能在数据库中执行任意 SQL 代码的攻击方式。这将导致记录被删除或泄露。
> Django 的 querysets 在被参数化查询构建出来时就被保护而免于 SQL 注入。查询的 SQL 代码与查询的参数是分开定义的。参数可能来自用户从而不安全，因此它们由底层数据库引擎进行转义。
> Django 也为开发者提供了书写 raw queries 或执行 custom sql 的权利。应当尽可能少地使用这些方法，并且您应该小心并准确的转义一切用户可控的参数。另外，在使用 extra() 和 RawSQL 时应当小心谨慎。
> 参数化查询在Django的ORM中无处不在，因此对于SQLi具有很强的防御能力，但是，在一些情况下，还是得注意注入，极少数的API并非100%安全。

有时django ORM不足以满足业务复杂度需求，需要使用原生SQL。不过在此之前，还请考虑是否有避免这种情况的方法。当然某些情况下，无法避免原生SQL，虽说可以通过一些django API来实现，不过不够保证安全。

### 执行原生查询
可以使用 Manager.raw() 来执行原生查询并返回模型实例，如下：
```
Manager.raw(raw_query, params=None, translations=None)¶
```
该方法接受一个原生 SQL 查询语句，执行它，并返回一个 django.db.models.query.RawQuerySet 实例。这个 RawQuerySet 能像普通的 QuerySet 一样被迭代获取对象实例。
例如：
![20210318-02](/img/20210318-02.png)

对于具体用法我们不过多说明，这里重点强调的是将参数传给raw(),可以使用 raw() 的 params 参数:
```
>>> lname = 'Doe'
>>> Person.objects.raw('SELECT * FROM myapp_person WHERE last_name = %s', [lname])
```
params 是一个参数字典。你将用一个列表替换查询字符串中 %s 占位符，或用字典替换 %(key)s 占位符（key 被字典 key 替换），不论你使用哪个数据库引擎。这些占位符会被 params 参数的值替换。

特别强调一下，不要对原生查询或SQL字符串中的引号占位符使用字符串格式化，反例如下：
```
>>> query = 'SELECT * FROM myapp_person WHERE last_name = %s' % lname
>>> Person.objects.raw(query)
```
这样相当于直接将lname参数拼接到sql语句中，进而导致了sql注入

### 直接执行自定义SQL

> 有时候，甚至 Manager.raw() 都无法满足需求：你可能要执行不明确映射至模型的查询语句，或者就是直接执行 UPDATE， INSERT 或 DELETE 语句。
> 这些情况下，你总是能直接访问数据库，完全绕过模型层。
> 对象 django.db.connection 代表默认数据库连接。要使用这个数据库连接，调用 connection.cursor() 来获取一个指针对象。然后，调用 cursor.execute(sql, [params]) 来执行该 SQL 和 cursor.fetchone()，或 cursor.fetchall() 获取结果数据
以上是官方wiki中说明的直接执行自定义SQL方法，对于参数的传递同样使用占位符的形式，wiki中并未提及extra方法。
```
extra(select=None, where=None, params=None, tables=None, order_by=None, select_params=None)
```
extra()函数提供QuerySet修改机制，它能在 QuerySet生成的SQL从句中注入新子句。

extra可以指定一个或多个 参数,例如 select, where or tables. 这些参数都不是必须的，但是你至少要使用一个，研发前辈给的建议是，要注意这些额外的方式对不同的数据库引擎可能存在移植性问题(因为你在显式的书写SQL语句),除非万不得已,尽量避免这样做。如果在审计过程中遇到，一定打起十二分的精神去分析。

## 缺陷代码分析
通过url的映射关系，找到了文件上传的入口。
![20210318-01](/img/20210318-01.png)
跟踪Upload对象，查看post方法实现
![20210318-03](/img/20210318-03.png)
分析文件上传post方法接口，第338行获取上传文件对象，第342行将上传文件文件名赋值给server_file_name，进过文件类型和空文件判断后，调用351行，拼接sql查询语句，调用第353行查询当前文件名是否存在，主要目的是文件重复判断。由于使用django objects.raw这种可以直接使用原生sql查询的方式，通过拼接sql语句并且系统中未设置任何sql过滤器，所以此处存在sql注入风险。

### JSON 函数和操作符
看到下面这条语句，第一反应是肯定有问题，但是对postgresql不怎么了解，对其sql语法也是一知半解，以下只是个人总结记录，看官请自行判断。
```
sql = "select * from %s.honey_trap_template where (trap_info::json#>>'{file_name}')::text = '%s'" % (
                SCHEMA_NAME, server_file_name)
```
#>> 操作符是以文本形式获取在指定路径的 JSON 对象，例如：
```
'{"a":[1,2,3],"b":[4,5,6]}'::json#>>'{a,2}'
```
运行的结果为3

相当于直接使用json对象，来处理数据，对于 (trap_info::json#>>'{file_name}')::text ，将trap_info转换成json对象，获取file_name数值，并将file_name参数格式化成text。

## 漏洞验证
使用burpsuite构建数据包进行测试，通过修改filename参数，确定注入点（主要是我前期构造数据包一直以为name对应代码中的的文件名，发现不起作用才发现自己找错地方了，暴露了自己垃圾测试功底）。
![20210318-04](/img/20210318-04.png)
通过提交恶意参数，使系统抛出异常。
![20210318-05](/img/20210318-05.png)
需要注意一点儿，系统会先对文件名的后缀（文件类型）进行判断，所以构建在payload及数据包，需要保留合法的后缀（文件类型），这里有zip、office系列。这里有一个基于时间延迟的payload，供大家学习
```
';SELECT PG_SLEEP(5)--.zip
```
sqlmap运行截图
![20210318-06](/img/20210318-06.png)

## 后记
该接口除了sql注入，同样对上传文件大小的上限未做限制，通过恶意上传大文件，可导致磁盘存储满载，进而导致拒绝服务。

## 参考
- [https://docs.djangoproject.com/zh-hans/3.1/topics/db/sql/](https://docs.djangoproject.com/zh-hans/3.1/topics/db/sql/)【执行原生sql查询】
- [http://www.postgres.cn/docs/10/functions-json.html](http://www.postgres.cn/docs/10/functions-json.html)【 JSON 函数和操作符】
