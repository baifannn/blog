---
title: Python05-异步IO
date: 2020-03-19 21:28:50
tags:
- python
categories:
- python学习
---
### 异步IO
在IO编程一节中，我们已经知道，CPU的速度远远快于磁盘、网络等IO。在一个线程中，CPU执行代码的速度极快，然而，一旦遇到IO操作，如读写文件、发送网络数据时，就需要等待IO操作完成，才能继续进行下一步操作。这种情况称为同步IO。

在IO操作的过程中，当前线程被挂起，而其他需要CPU执行的代码就无法被当前线程执行了。

因为一个IO操作就阻塞了当前线程，导致其他代码无法执行，所以我们必须使用多线程或者多进程来并发执行代码，为多个用户服务。每个用户都会分配一个线程，如果遇到IO导致线程被挂起，其他用户的线程不受影响。

多线程和多进程的模型虽然解决了并发问题，但是系统不能无上限地增加线程。由于系统切换线程的开销也很大，所以，一旦线程数量过多，CPU的时间就花在线程切换上了，真正运行代码的时间就少了，结果导致性能严重下降。

由于我们要解决的问题是CPU高速执行能力和IO设备的龟速严重不匹配，多线程和多进程只是解决这一问题的一种方法。

另一种解决IO问题的方法是异步IO。当代码需要执行一个耗时的IO操作时，它只发出IO指令，并不等待IO结果，然后就去执行其他代码了。一段时间后，当IO返回结果时，再通知CPU进行处理。

可以想象如果按普通顺序写出的代码实际上是没法完成异步IO的：
```python
do_some_code()
f = open('/path/to/file', 'r')
r = f.read() # <== 线程停在此处等待IO操作结果
# IO操作完成后线程才能继续执行:
do_some_code(r)
```

所以，同步IO模型的代码是无法实现异步IO模型的。

### 协程

程，又称微线程，纤程。英文名Coroutine。

协程的概念很早就提出来了，但直到最近几年才在某些语言（如Lua）中得到广泛应用。

子程序，或者称为函数，在所有语言中都是层级调用，比如A调用B，B在执行过程中又调用了C，C执行完毕返回，B执行完毕返回，最后是A执行完毕。

所以子程序调用是通过栈实现的，一个线程就是执行一个子程序。

子程序调用总是一个入口，一次返回，调用顺序是明确的。而协程的调用和子程序不同。

协程看上去也是子程序，但执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。

注意，在一个子程序中中断，去执行其他子程序，不是函数调用，有点类似CPU的中断。比如子程序A、B：

```python
def A():
    print('1')
    print('2')
    print('3')

def B():
    print('x')
    print('y')
    print('z')
```

假设由协程执行，在执行A的过程中，可以随时中断，去执行B，B也可能在执行过程中中断再去执行A，结果可能是：

```
1
2
x
y
3
z
```

但是在A中是没有调用B的，所以协程的调用比函数调用理解起来要难一些。

看起来A、B的执行有点像多线程，但协程的特点在于是一个线程执行，那和多线程比，协程有何优势？

最大的优势就是协程极高的执行效率。因为子程序切换不是线程切换，而是由程序自身控制，因此，没有线程切换的开销，和多线程比，线程数量越多，协程的性能优势就越明显。

第二大优势就是不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。

因为协程是一个线程执行，那怎么利用多核CPU呢？最简单的方法是多进程+协程，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。

Python对协程的支持是通过generator实现的。

在generator中，我们不但可以通过for循环来迭代，还可以不断调用next()函数获取由yield语句返回的下一个值。

但是Python的yield不但可以返回一个值，它还可以接收调用者发出的参数。

来看例子：

传统的生产者-消费者模型是一个线程写消息，一个线程取消息，通过锁机制控制队列和等待，但一不小心就可能死锁。

如果改用协程，生产者生产消息后，直接通过yield跳转到消费者开始执行，待消费者执行完毕后，切换回生产者继续生产，效率极高：

```python
def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

def produce(c):
    c.send(None)
    # 启动生成器 next(c)    
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()

c = consumer()
produce(c)
```

执行结果：

```
[PRODUCER] Producing 1...
[CONSUMER] Consuming 1...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 2...
[CONSUMER] Consuming 2...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 3...
[CONSUMER] Consuming 3...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 4...
[CONSUMER] Consuming 4...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 5...
[CONSUMER] Consuming 5...
[PRODUCER] Consumer return: 200 OK
```

注意到consumer函数是一个generator，把一个consumer传入produce后：

1. 首先调用c.send(None)启动生成器；
2. 然后，一旦生产了东西，通过c.send(n)切换到consumer执行；
3. consumer通过yield拿到消息，处理，又通过yield把结果传回；
4. `produce`拿到`consumer`处理的结果，继续生产下一条消息；
5. produce决定不生产了，通过c.close()关闭consumer，整个过程结束。

整个流程无锁，由一个线程执行，produce和consumer协作完成任务，所以称为“协程”，而非线程的抢占式多任务。

最后套用Donald Knuth的一句话总结协程的特点：

“子程序就是协程的一种特例。”

#### 详解

想放弃的我又回来了，额...

要看懂上面的例子，关键在于要理解下面几点：

1、例子中的`c.send(None)`，其功能类似于`next(c)`，比如：

```python
>>> def num():
        yield 1
        yield 2
    
>>> c = num()
>>> c.send(None)
1
>>> c.send(None)
2
>>> c.send(None)
Traceback (most recent call last):
  File "<pyshell#9>", line 1, in <module>
    c.send(None)
StopIteration
>>> 
```

2、`n = yield r`，这里是一条语句，但要理解两个知识点，赋值语句先计算`=` 右边，由于右边是 `yield` 语句，所以`yield`语句执行完以后，进入暂停，而赋值语句在下一次启动生成器的时候首先被执行；

3、`send` 在接受`None`参数的情况下，等同于`next(generator)`的功能，但`send`同时也可接收其他参数，比如例子中的`c.send(n)`，要理解这种用法，先看一个例子：

```python
>>> def num():
        a = yield 1
        while True:
            a = yield a
       
>>> c = num()
>>> c.send(None)
1
>>> c.send(5)
5
>>> c.send(100)
100
```

在上面的例子中，首先使用 `c.send(None)`，返回生成器的第一个值，`a = yield 1` ，也就是`1`（但此时，并未执行赋值语句），

接着我们使用了`c.send(5)`，再次启动生成器，并同时传入了一个参数`5`，再次启动生成的时候，从上次`yield`语句断掉的地方开始执行，即 `a` 的赋值语句，由于我们传入了一个参数`5`，所以`a`被赋值为5，接着程序进入`whlie`循环，当程序执行到 `a = yield a`，同理，先返回生成器的值 `5`，下次启动生成器的时候，再执行赋值语句，以此类推...

所以`c.send(n)`的用法就是老师上文中所说的 ，" Python的`yield`不但可以返回一个值，它还可以接收调用者发出的参数。"

**但注意，在一个生成器函数未启动之前，是不能传递值进去。也就是说在使用`c.send(n)`之前，必须先使用`c.send(None)`或者`next(c)`来返回生成器的第一个值。**

最后我们来看上文中的例子，梳理下执行过程：

```python
def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

def produce(c):
    c.send(None)
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()

c = consumer()
produce(c)
```

第一步：执行 `c.send(None)`，启动生成器返回第一个值，`n = yield r`，此时 `r` 为空，`n` 还未赋值，然后生成器暂停，等待下一次启动。

第二步：生成器返回空值后进入暂停，`produce(c)` 接着往下运行，进入`While`循环，此时 `n` 为`1`，所以打印：

```
[PRODUCER] Producing 1...
```

第三步：`produce(c)` 往下运行到 `r = c.send(1)`，再次启动生成器，并传入了参数`1`，而生成器从上次`n`的赋值语句开始执行，`n` 被赋值为`1`，`n`存在，`if` 语句不执行，然后打印：

```
[CONSUMER] Consuming 1...
```

接着`r`被赋值为`'200 OK'`，然后又进入循环，执行`n = yield r`，返回生成器的第二个值，`'200 OK'`，然后生成器进入暂停，等待下一次启动。

第四步：生成器返回`'200 OK'`进入暂停后，`produce(c)`往下运行，进入`r`的赋值语句，`r`被赋值为`'200 OK'`，接着往下运行，打印：

```
[PRODUCER] Consumer return: 200 OK
```

以此类推...

当`n`为`5`跳出循环后，使用`c.close()` 结束生成器的生命周期，然后程序运行结束。

### asyncio

我们用Task封装两个`coroutine`试试：

```python
import threading
import asyncio

@asyncio.coroutine
def hello():
    print('Hello world! (%s)' % threading.currentThread())
    yield from asyncio.sleep(1)
    print('Hello again! (%s)' % threading.currentThread())

loop = asyncio.get_event_loop()
tasks = [hello(), hello()]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

观察执行过程：

```
Hello world! (<_MainThread(MainThread, started 140735195337472)>)
Hello world! (<_MainThread(MainThread, started 140735195337472)>)
(暂停约1秒)
Hello again! (<_MainThread(MainThread, started 140735195337472)>)
Hello again! (<_MainThread(MainThread, started 140735195337472)>)
```

由打印的当前线程名称可以看出，两个`coroutine`是由同一个线程并发执行的。

如果把`asyncio.sleep()`换成真正的IO操作，则多个`coroutine`就可以由一个线程并发执行。

我们用`asyncio`的异步网络连接来获取sina、sohu和163的网站首页：

```python
import asyncio

@asyncio.coroutine
def wget(host):
    print('wget %s...' % host)
    connect = asyncio.open_connection(host, 80)
    reader, writer = yield from connect
    header = 'GET / HTTP/1.0\r\nHost: %s\r\n\r\n' % host
    writer.write(header.encode('utf-8'))
    yield from writer.drain()
    while True:
        line = yield from reader.readline()
        if line == b'\r\n':
            break
        print('%s header > %s' % (host, line.decode('utf-8').rstrip()))
    # Ignore the body, close the socket
    writer.close()

loop = asyncio.get_event_loop()
tasks = [wget(host) for host in ['www.sina.com.cn', 'www.sohu.com', 'www.163.com']]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

执行结果如下：

```
wget www.sohu.com...
wget www.sina.com.cn...
wget www.163.com...
(等待一段时间)
(打印出sohu的header)
www.sohu.com header > HTTP/1.1 200 OK
www.sohu.com header > Content-Type: text/html
...
(打印出sina的header)
www.sina.com.cn header > HTTP/1.1 200 OK
www.sina.com.cn header > Date: Wed, 20 May 2015 04:56:33 GMT
...
(打印出163的header)
www.163.com header > HTTP/1.0 302 Moved Temporarily
www.163.com header > Server: Cdn Cache Server V2.0
...
```

可见3个连接由一个线程通过`coroutine`并发完成。

### async/await

用`asyncio`提供的`@asyncio.coroutine`可以把一个generator标记为coroutine类型，然后在coroutine内部用`yield from`调用另一个coroutine实现异步操作。

为了简化并更好地标识异步IO，从Python 3.5开始引入了新的语法`async`和`await`，可以让coroutine的代码更简洁易读。

请注意，`async`和`await`是针对coroutine的新语法，要使用新的语法，只需要做两步简单的替换：

1. 把`@asyncio.coroutine`替换为`async`；
2. 把`yield from`替换为`await`。

让我们对比一下上一节的代码：

```python
@asyncio.coroutine
def hello():
    print("Hello world!")
    r = yield from asyncio.sleep(1)
    print("Hello again!")
```

用新语法重新编写如下：

```python
async def hello():
    print("Hello world!")
    r = await asyncio.sleep(1)
    print("Hello again!")
```

剩下的代码保持不变。

### aiohttp

`asyncio`可以实现单线程并发IO操作。如果仅用在客户端，发挥的威力不大。如果把`asyncio`用在服务器端，例如Web服务器，由于HTTP连接就是IO操作，因此可以用单线程+`coroutine`实现多用户的高并发支持。

`asyncio`实现了TCP、UDP、SSL等协议，`aiohttp`则是基于`asyncio`实现的HTTP框架。

我们先安装`aiohttp`：

```bash
pip install aiohttp
```

然后编写一个HTTP服务器，分别处理以下URL：

- `/` - 首页返回`b'Index'`；
- `/hello/{name}` - 根据URL参数返回文本`hello, %s!`。

代码如下：

```python
import asyncio

from aiohttp import web

async def index(request):
    await asyncio.sleep(0.5)
    return web.Response(body=b'<h1>Index</h1>')

async def hello(request):
    await asyncio.sleep(0.5)
    text = '<h1>hello, %s!</h1>' % request.match_info['name']
    return web.Response(body=text.encode('utf-8'))

async def init(loop):
    app = web.Application(loop=loop)
    app.router.add_route('GET', '/', index)
    app.router.add_route('GET', '/hello/{name}', hello)
    srv = await loop.create_server(app.make_handler(), '127.0.0.1', 8000)
    print('Server started at http://127.0.0.1:8000...')
    return srv

loop = asyncio.get_event_loop()
loop.run_until_complete(init(loop))
loop.run_forever()
```

注意`aiohttp`的初始化函数`init()`也是一个`coroutine`，`loop.create_server()`则利用`asyncio`创建TCP服务。