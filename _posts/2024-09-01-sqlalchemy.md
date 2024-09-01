---
layout: post
title: sqlalchemy写数据结构化异常处理
categories: [Sqlalchemy, 数据库, ORM]
description: Sqlalchemy, 异常处理, sql
keywords: Sqlalchemy, ORM, 异常处理, sql
---

## 问题现象

最近在备份数据表的时候，由于想把导出的sql文件用Sqlalchemy库写入不同的schema，所以使用了如下的代码

```python

from sqlalchemy import text
    with open ('test.sql', 'r') as f:
        session.execute(text(f))
        session.commit()    
    
```

但是却有报错

```python

报错信息待补充

```

查看报错信息说的是想要插入一个float值但是数据并不合法，于是开始排查数据库表结构，发现并没有float类型的字段，为什么会有这个报错呢。
再看一下错误信息的字段，想要插入 `N'{?=6}'` 的数据，但是查看sql文件的插入数据，原始数据其实是 `N'{:0=6}'`，为什么数据被改变了呢，于是开始排查。


## 错误原因

1. 第一步，是否是因为`text()`函数的封装让原始的sql语句发生了改变，测试发现并没有
2. 第二步，直接执行原始的文件中的语句，还是相同的报错
3. 第三步，直接对数据库执行相同的sql插入语句，发现没有问题

结论是Sqlalchemy的session.excute函数在执行目标sql语句时做了相关的修改，于是查阅资料，是orm库在执行sql语句时支持结构化字符串的输入，`:`  `?` 这两个字符都有特殊的含义，会导致原生的语句被转义，简单说明如下

### Sqlalchemy 参数使用说明

- 命名参数 ":"

Sqlalchemy中的 connection.execute方法支持命名参数来传递参数。在SQL语句中使用冒号和变量名的形式即可，示例如下

```python
from sqlalchemy import create_engine
from sqlalchemy.sql import text

engine = create_engine('conn addr')
conn = engine.connect()

sql = text('SELECT * FROM users WHERE age > :age')
result = conn.execute(sql, age=18)

```

这个时候age就可以作为参数传进去



- 占位符 "?"

占位符是一个问号，表示参数的位置，在执行sql语句时，可以将参数的值作为一个列表或者元组传递给connection.execute方法，示例如下

```python
from sqlalchemy import create_engine
from sqlalchemy.sql import text

engine = create_engine('conn addr')
conn = engine.connect()

sql = text('SELECT * FROM users WHERE age > ?')
result = conn.execute(sql, [18])
result = conn.execute(sql, (20,))
```
这个时候就可以把参数值传进去

- 两个占位符当然也可以一起使用

```python
sql = text('SELECT * FROM users WHERE age > :age AND name = ?')

result = conn.execute(sql, age=18, 'test')
```

------
看了上面的说明也就能知道为什么会有之前的报错了，因为原始的插入语句中`N'{:0=6}'` 问号后面的`0`被识别成了一个名字为`0`的变量，所以Sqlalchemy在期待输入这个变量的值而导致的异常。


## 触发场景

使用 `connection.execute` 或者 `session.execute`时，sql语句中有占位符 `?`或者`:`

## 如何解决

- 使用orm的model写入数据，在使用model定义的对象写入数据时，值将不作任何处理直接保存，所以也不存在转义的错误

- 使用 raw_connection，orm框架提供的这个方法将会绕过orm而直接对底层的sql驱动发送sql语句，所以也就不存在占位符的错误解析了

- 使用示例

```python

connection = engine.raw_connection()
cursor = connection.cursor()
cursor.execute("your sql")
rows = cursor.fetchall()

cursor.close()
connection.close()
```

## 相关源码解读

