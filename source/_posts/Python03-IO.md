---
title: Python03-IO
date: 2020-03-18 22:16:50
tags:
- python
categories:
- python学习
---

### 读文件

```python
f=open('/User/baifan/text.txt',mode='r')
```

mode类型

| 文件打开模式 | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| r            | 以只读模式打开文件，并将文件指针指向文件头；如果文件不存在会报错 |
| w            | 以只写模式打开文件，并将文件指针指向文件头；如果文件存在则将其内容清空，如果文件不存在则创建 |
| a            | 以只追加可写模式打开文件，并将文件指针指向文件尾部；如果文件不存在则创建 |
| r+           | 在r的基础上增加了可写功能                                    |
| w+           | 在w的基础上增加了可读功能                                    |
| a+           | 在a的基础上增加了可读功能                                    |
| b            | 读写二进制文件（默认是t，表示文本），需要与上面几种模式搭配使用，如ab，wb, ab, ab+（POSIX系统，包括Linux都会忽略该字符） |

- r+会覆盖当前文件指针所在位置的字符，如原来文件内容是"Hello，World"，打开文件后写入"hi"则文件内容会变成"hillo, World"
- w+与r+的不同是，w+在打开文件时就会先将文件内容清空，不知道它有什么用
- a+与r+的不同是，a+只能写到文件末尾（无论当前文件指针在哪里）

如果打开文件成功，使用下面命令读取：

```python
f.read()
#output:
#'hello world'
```

下面调用close进行关闭

```python
f.close()
```

调用read()会一次性读取文件的全部内容，如果文件有10G，内存就爆了，所以，要保险起见，可以反复调用read(size)方法，每次最多读取size个字节的内容。另外，调用readline()可以每次读取一行内容，调用readlines()一次读取所有内容并按行返回list。因此，要根据需要决定怎么调用。

如果文件很小，read()一次性读取最方便；如果不能确定文件大小，反复调用read(size)比较保险；如果是配置文件，调用readlines()最方便：

```python
for line in f.readlines():
    print(line.strip()) # 把末尾的'\n'删掉
```



### 字符编码

要读取非UTF-8编码的文本文件，需要给open()函数传入encoding参数，例如，读取GBK编码的文件：

```python
f = open('/Users/michael/gbk.txt', 'r', encoding='gbk')
f.read()
'测试'
```

遇到有些编码不规范的文件，你可能会遇到UnicodeDecodeError，因为在文本文件中可能夹杂了一些非法编码的字符。遇到这种情况，open()函数还接收一个errors参数，表示如果遇到编码错误后如何处理。最简单的方式是直接忽略：

```python
f = open('/Users/michael/gbk.txt', 'r', encoding='gbk', errors='ignore')
```



### 写文件

写文件和读文件是一样的，唯一区别是调用open()函数时，传入标识符'w'或者'wb'表示写文本文件或写二进制文件：

```python
f = open('/Users/michael/test.txt', 'w')
f.write('Hello, world!')
f.close()
```

你可以反复调用write()来写入文件，但是务必要调用f.close()来关闭文件。当我们写文件时，操作系统往往不会立刻把数据写入磁盘，而是放到内存缓存起来，空闲的时候再慢慢写入。只有调用close()方法时，操作系统才保证把没有写入的数据全部写入磁盘。忘记调用close()的后果是数据可能只写了一部分到磁盘，剩下的丢失了。所以，还是用with语句来得保险：

```python
with open('/Users/michael/test.txt', 'w') as f:
    f.write('Hello, world!')
```

要写入特定编码的文本文件，请给open()函数传入encoding参数，将字符串自动转换成指定编码。



### 操作文件和目录

使用os模块

```python
import os 
#操作系统类型
os.name
#系统信息
os.uname()
#环境变量
os.environ
```

使用os和os.path操作目录：

```python
# 查看当前目录的绝对路径:
>>> os.path.abspath('.')
'/Users/baifan'
# 在某个目录下创建一个新目录，首先把新目录的完整路径表示出来:
>>> os.path.join('/Users/baifan', 'testdir')
'/Users/baifan/testdir'
# 然后创建一个目录:
>>> os.mkdir('/Users/baifan/testdir')
# 删掉一个目录:
>>> os.rmdir('/Users/baifan/testdir')
```



### 序列化 json

如果我们要在不同的编程语言之间传递对象，就必须把对象序列化为标准格式，比如XML，但更好的方法是序列化为JSON，因为JSON表示出来就是一个字符串，可以被所有语言读取，也可以方便地存储到磁盘或者通过网络传输。JSON不仅是标准格式，并且比XML更快，而且可以直接在Web页面中读取，非常方便。

JSON表示的对象就是标准的JavaScript语言的对象，JSON和Python内置的数据类型对应如下：

| JSON类型   | Python类型 |
| :--------- | :--------- |
| {}         | dict       |
| []         | list       |
| "string"   | str        |
| 1234.56    | int或float |
| true/false | True/False |
| null       | None       |

Python内置的json模块提供了非常完善的Python对象到JSON格式的转换。我们先看看如何把Python对象变成一个JSON：

```python
>>> import json
>>> d = dict(name='Bob', age=20, score=88)
>>> json.dumps(d)
'{"age": 20, "score": 88, "name": "Bob"}'
```

dumps()方法返回一个str，内容就是标准的JSON。类似的，dump()方法可以直接把JSON写入一个file-like Object。

要把JSON反序列化为Python对象，用loads()或者对应的load()方法，前者把JSON的字符串反序列化，后者从file-like Object中读取字符串并反序列化：

```python
>>> json_str = '{"age": 20, "score": 88, "name": "Bob"}'
>>> json.loads(json_str)
{'age': 20, 'score': 88, 'name': 'Bob'}
```

由于JSON标准规定JSON编码是UTF-8，所以我们总是能正确地在Python的str与JSON的字符串之间转换。